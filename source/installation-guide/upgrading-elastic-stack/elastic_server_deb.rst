.. Copyright (C) 2019 Wazuh, Inc.

.. _elastic_server_deb_legacy:

Upgrading Elastic Stack with DEB packages
=========================================

Prepare the Elastic Stack
-------------------------

1. Stop the services:

  .. code-block:: console

    # systemctl stop filebeat
    # systemctl stop logstash
    # systemctl stop kibana
    # systemctl stop elasticsearch

2. Add the new repository for Elastic Stack 7.x:

  .. code-block:: console

    # curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
    # echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list

Upgrade Elasticsearch
---------------------

1. Disable shard allocation

  .. code-block:: bash

    curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
    {
      "persistent": {
        "cluster.routing.allocation.enable": "primaries"
      }
    }
    '

2. Stop non-essential indexing and perform a synced flush. (Optional)

  .. code-block:: bash

    curl -X POST "localhost:9200/_flush/synced"

3. Shut down a single node.

  .. code-block:: console
    
    # systemctl stop elasticsearch.service

4. Upgrade the node you shut down.

  .. code-block:: console
    
    # apt-get install elasticsearch=7.0.1
    # systemctl restart elasticsearch

5. Start the newly-upgraded node and confirm that it joins the cluster by checking the log file or by submitting a _cat/nodes request:

  .. code-block:: bash

    curl -X GET "localhost:9200/_cat/nodes"

6. Reenable shard allocation.

  .. code-block:: bash

    curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
    {
      "persistent": {
        "cluster.routing.allocation.enable": null
      }
    }
    '

7. Before upgrading the next node, wait for the cluster to finish shard allocation. 

  .. code-block:: bash

    curl -X GET "localhost:9200/_cat/health?v"

8. Repeat it for every Elasticsearch node.

Upgrade Filebeat
----------------

1. Update the configuration file.

  .. code-block:: console

    # cp /etc/filebeat/filebeat.yml /backup/filebeat.yml.backup
    # curl -so /etc/filebeat/filebeat.yml https://raw.githubusercontent.com/wazuh/wazuh/3.9/extensions/filebeat/filebeat.yml

2. Upgrade Filebeat.

  .. code-block:: console

    # apt-get install filebeat=7.0.1

3. Restart Filebeat.

  .. code-block:: console

    # systemctl restart filebeat

Remove Logstash
---------------

Since Elastic 7.0 both Logstash and Java are no longer needed. Filebeat will do the job with our new configuration.

Upgrade Kibana
--------------

1. Since Kibana 7.0.1, the Elasticsearch server address setting has been changed, if your Elasticsearch is not on ``localhost``, please replace ``elasticsearch.url: "address:9200"`` with ``elasticsearch.hosts: ["address:9200"]``.
2. Remove the Wazuh app.

  .. code-block:: console

    # /usr/share/kibana/bin/kibana-plugin remove wazuh

3. Upgrade Kibana.

  .. code-block:: console

    # apt-get install kibana=7.0.1

4. Install the Wazuh app.

  .. code-block:: console

    # sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp-3.9.0_7.0.1.zip

5. Restart Kibana.

  .. code-block:: console

    # systemctl restart kibana

