.. _architecture:

Architecture
============

When monitoring a network, the Wazuh architecture consists in multiple agents running on the monitoring hosts and reporting to a central manager. This central manager is also sending data, resulting from the analysis done by decoders and rules, to an Elastic Stack cluster.

Elasticsearch clusters are a collection of nodes (systems) that communicate with each other to read and write to an index. Small Wazuh deployments (<50 agents), can easily be handled by a single-node cluster. Multi-node clusters are recommended when there is a large number of monitored systems or large volume of data is planned to be indexed and stored.

The integration between a Wazuh manager and an Elasticsearch cluster is done by configuring Filebeat to read alerts and archived events, forwarding the data to the Logstash server, which resides on the Elasticsearch cluster. Filebeat can be configured to use TLS encryption when talking to Logstash.

See below how components are distributed when the Wazuh manager and the Elastic Stack cluster run in different hosts:

.. thumbnail:: ../images/installation/installing_wazuh.png
    :title: Distributed architecture 
    :align: center
    :width: 100%

For even smaller deployments both, the Wazuh manager and the Elastic Stack single-node cluster, can run in the same host:

.. thumbnail:: ../images/installation/installing_wazuh_singlehost.png
    :title: Single-host architecture
    :align: center
    :width: 100%

Data flow
---------

.. thumbnail:: ../images/data_flow_2048x794.png
    :title: Data flow
    :align: center
    :width: 100%


Agent-manager communication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Wazuh agents use OSSEC messages protocol to communicate with the Wazuh manager (port 1514/UDP), in order to send collected information. Once OSSEC manager receives data messages, those are processed through the analysis engine, meaning that some of those messages generate alerts. Alerts and raw log messages are stored into different files:

 - The file */var/ossec/logs/archives/archives.json* contains the raw logs.
 - The file */var/ossec/logs/alerts/alerts.json* contains the alerts.

OSSEC protocol encrypts messages using Blowfish with a 192 bits encryption key with 16-round implementation.


Manager-elastic communication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Wazuh manager uses Filebeat to ship alerts and events data to Logstash (5000/TCP). The communication is done via SSL. For a single-host arquitecture, Logstash is able to read the alerts directly from the Manager without use Filebeat.

Internally Logstash formats the data and send it to Elasticsearch (port 9200/TCP). Once the data is indexed into Elasticsearch, Kibana (port 5601/TCP) is used to visualice the information.

Wazuh App runs inside Kibana and execute queries against the RESTful API (Manager - port 55000/TCP) in order to get information related to the configuration and status of the manager and agents. These queries are run over SSL for encryption, and with username/password authentication.


Data storage
-----------------------------

Alerts and events (raw logs) data is stored both at the Wazuh manager and Elastic Stack levels, meaning that the manager keeps a copy of the logs sent to the Elastic Stack cluster.
This data is compressed and signed using MD5 and SHA1 checksums. Files are rotated on daily basis, following this structure:

.. code-block:: console

    root@vpc-ossec-manager:/var/ossec/logs/archives/2017/Jan# ls -l
    total 176
    -rw-r----- 1 ossec ossec 350 Jan  2 00:00 ossec-archive-01.json.sum
    -rw-r----- 1 ossec ossec 346 Jan  2 00:00 ossec-archive-01.log.sum
    -rw-r----- 1 ossec ossec 350 Jan  3 00:01 ossec-archive-02.json.sum
    -rw-r----- 1 ossec ossec 346 Jan  3 00:01 ossec-archive-02.log.sum
    -rw-r----- 1 ossec ossec 350 Jan  4 00:00 ossec-archive-03.json.sum
    -rw-r----- 1 ossec ossec 346 Jan  4 00:00 ossec-archive-03.log.sum
    -rw-r----- 1 ossec ossec 350 Jan  5 00:01 ossec-archive-04.json.sum
    -rw-r----- 1 ossec ossec 346 Jan  5 00:01 ossec-archive-04.log.sum

Depending on the storage capacity, rotation of data and backup is recommended, running *cron* tasks to keep only a certain period of time locally stored on the manager (e.g. last year or last three months).

On the other hand, once an instance of an event data is processed by Elasticsearch it is stored in the form of an Apache Lucene inverted index, which can be composed by multiple segments split in different files. String data is encoded using UTF-8.

In order to access indexed data Kibana uses Elasticsearch REST queries. It is recommended to periodically create Elasticsearch snapshots, to backup data. A *cron* task can also be used in this case to move it to a final data storage server, and sign files using MD5 and SHA1 algorithms.
