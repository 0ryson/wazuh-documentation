Integrating OSSEC-ELK Stack
=============================================================

Introduction
--------------------
Welcome to OSSEC-ELK Stack integration guide by Wazuh, throught some simple steps you will set up an entire ELK Stack architecture to monitor, collect, process, index and display your OSSEC Alerts.

The full guide is based on OSSEC Wazuh version, we contribute with OSSEC community by developing extended funcionality.

These are some features we are talking about:

* **OSSEC-Wazuh Ruleset**
   Weekly OSSEC Ruleset updates
* **OSSEC PCI DSS 3.0 & CIS Requirements**
   Detailed groups and Benchmarks (ex: 1.4 Debian Linux Benchmark, 11.4 PCI...)
* **OSSEC JSON Custom Output**
   Groups array, timestamps, Agents name, locations, file integrity..
* **Logstash input/filter/output plugins**
   GeoIP, names format, elasticsearch template, elasticsearch ossec cluster.
* **Elasticsearch**
   Custom index mapping template to fit OSSEC alert fields. 
* **Kibana 4**
   OSSEC Alerts, PCI Complianace, CIS Compliance, Agents management, Agents Info Dashboards.
   Hiding non useful fields, display short summary of PCI Requirements on mouseover on PCI Alerts.

If you have any questions or issues while reading this guide don't hesitate to contact us at: info@wazuh.com


.. note:: This functionalty requires OSSEC-Wazuh version and OSSEC-Wazuh Ruleset, keep reading this guide to install it.


Architecture explanation
-------------------------
The whole architecture is based on log analysis, collect, index and display alerts. To acomplish this we are going to use the following tools:

**OSSEC-HIDS**

`OSSEC Official Website <http://www.ossec.net/>`_

OSSEC is an Open Source Host-based Intrusion Detection System that performs log analysis, file integrity checking, policy monitoring, rootkit detection, real-time alerting and active response.

**Logstash**

`Logstash Official Website <https://www.elastic.co/products/logstash/>`_

Logstash is a data pipeline that helps you process logs and other event data from a variety of systems. Logstash will read and process send our OSSEC JSON Files to Elasticsearch Cluster.

**Logstash-Forwarder**

`Logstash-Forwarder Official Website <https://www.elastic.co/products/logstash/>`_

Logstash-Forwarder is a shipment tool to ship logs from our Servers to our Logstash Server, in our case we will send logs from OSSEC Manager host to ELK Stack host.

**Elasticsearch**

`Elasticsearch Official Website <https://www.elastic.co/products/elasticsearch/>`_

Search & Analyze Data in Real Time. Distributed, scalable, and highly available. Real-time search and analytics capabilities. Elasticsearch will index and sotre all our OSSEC alerts, this way we will be able to search and explore thousands of alerts in few clicks.

**Kibana**

`Kibana Official Website <https://www.elastic.co/products/kibana/>`_

Kibana is a friendly WEB interface to explore all elasticsearch indexes, Kibana support custom dashboard creations, in our case Security Compliance dashboards and OSSEC high risk security alerts.

**Scaling to large deployments**

To multi-node and high availability architectures we will use some extra tools like Logstash-Forwarder or Redis Server.


Installing
-------------------------
Requirements
^^^^^^^^^^^^^^^^^^^
* SSH access to at least one server and sudo privilegies.
* Java 8 `(Example Ubuntu install guide) <http://tecadmin.net/install-oracle-java-8-jdk-8-ubuntu-via-ppa/>`_

Considerations
^^^^^^^^^^^^^^^^^^^
**Single/multiple host configurations**

The entire guide is orientated to single-node configuration but you still can build this architecture up on differentes servers, it is recommended to split ELK Server from OSSEC Manager server, for example, four differentes hosts: 

* Host 1: OSSEC Manager+Logstash Forwarder
* Host 2: Logstash server + Elasticsearch + Kibana
* Host 3: Node 2
* Host 3: Node 3

We will give you differents configuration files depends on the architecture you choose.

Remember

* Single-host: All the tools on same machine
* Multi-host: Tools split-up on differents machines.

1. OSSEC
^^^^^^^^^^^^^^^^^^^
First of all, download the whole OSSEC-Wazuh repository from Github which includes OSSEC HIDS latest version (2.8.2 base), Wazuh enhace capabilites and ELK Stack configuration files.

1.1 Installation
""""""""""""""""""

Create a folder on your prefered home directory and download the repository like this:

Go home folder, create tmp folder, clone the repository ::

   $ cd ~
   $ mkdir ossec_tmp && cd ossec_tmp
   $ git clone https://github.com/wazuh/ossec-wazuh.git
   $ cd ossec-wazuh

Now we have the OSSEC source code on our machine, let's compile it. We need development and packages tools like g++, gcc etc... if it is needed, install them.

Finally compile and install OSSEC Manager by entering ::

   $ sudo ./install

Follow the installation steps OSSEC prompts at console, they are identical to OSSEC official version, you can read a detailed explanation here: 

`Manager installation  <http://documentation.wazuh.com/en/latest/source.html#manager-installation/>`_

Remember we ARE NOT installing official OSSEC relealse, you need to compile and install Wazuh version.

You can let all prompt steps by **default** by pressing ENTER at every question OSSEC installation ask you, by now, we don't need a specific OSSEC config installation.


1.2 Configuration
""""""""""""""""""
We need just one tweak at OSSEC configuration files, enable JSON output. 

Open OSSEC conf file ::

   $ sudo vi /var/ossec/etc/ossec.conf

Add inside **<global></global>** tags the json output setting ::

   <jsonout_output>yes</jsonout_output>

That's all! Now restart your OSSEC Manager ::

   $ sudo /var/ossec/bin/ossec-control start

Check if alerts.json file exits and is working ::

   $ sudo cat /var/ossec/logs/alerts/alerts.json


1.3 Agents
""""""""""""""""""
This section is covered `here <http://documentation.wazuh.com/en/latest/source.html#agent-installation>`_

2. Logstash
^^^^^^^^^^^^^^^^^^^
.. note:: At this poing you will need Java 8 installed on your system, please proceed to install it before install any of next tools. `Install Java 8 <http://tecadmin.net/install-oracle-java-8-jdk-8-ubuntu-via-ppa/>`_

We proceed to install Logstash Server, in this case we are installing it on the **same** machine we previously installed OSSEC Manager, that's why some configuration settings will refer local OSSEC files.

2.1 Installation
""""""""""""""""""
**Logstash 1.5 version**

We recommend to install Logstash from official repositories, inside next link you will find YUM and DEB packages.

`Elastic.co: Install Logstash from repositories <https://www.elastic.co/guide/en/logstash/current/package-repositories.html>`_

For instance, to install DEB packages for example to an Ubuntu SO:

Download and install the Public Signing Key: ::

   $ wget -qO - https://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -

Add the repository definition to your /etc/apt/sources.list file: ::

   $ echo "deb http://packages.elasticsearch.org/logstash/1.5/debian stable main" | sudo tee -a /etc/apt/sources.list

Run sudo apt-get update and the repository is ready for use. You can install it with: ::

   $ sudo apt-get update && sudo apt-get install logstash
   

2.1 Configuration
""""""""""""""""""

**Configuration files**

Once Logstash be installed copy Wazuh **SINGLE-HOST** Logstash file to Logstash configuration files ::

  $ sudo cp ~/ossec_tmp/ossec-wazuh/extensions/logstash/01-ossec-singlehost.conf /etc/logstash/conf.d/

Or copy Wazuh Logstash **MULTI-HOST** file to Logstash configuration files ::

  $ sudo cp ~/ossec_tmp/ossec-wazuh/extensions/logstash/01-ossec.conf  /etc/logstash/conf.d/

In this case don't forget to edit 01-ossec.conf file to replace your Elasicsearch destination IP ::

  elasticsearch {
           host => "your_elasticsearch_server_ip"


**GeoIP DB** 

Download GeoLiteCity from Maxmind website, unzip and move to Logstash folder ::

  $ sudo curl -O "http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz"
  $ sudo sudo gzip -d GeoLiteCity.dat.gz && sudo mv GeoLiteCity.dat /etc/logstash/

**Logstash user** 

In case you are installing single-host architecture, Logstash will need to read from OSSEC alerts file, we need to grants permission to do that.

Open users groups file ::

  $ sudo vi /etc/group

Search for "ossec" and add logstash at the end of that line, just like this ::

  ubuntu:x:1000:
  ossec:x:1001:logstash
  scanner:x:111:

**Restart Logstash** 
  
Finally restart Logstash service to apply last changes ::

 $ sudo service logstash restart

3. Logstash-Forwarder
^^^^^^^^^^^^^^^^^^^

.. warning:: Logstash-Forwarder configuration it is only neccesary to **multi-host** architecture, if you are installing all tools on one machine, you don't need to install Logstash-Forwarder, please refer directly to secction `3. Elasticsearch <#id3>`_


3.1 Generate SSL Certificates on Logstash-Server
""""""""""""""""""
Since we are going to use Logstash Forwarder to ship logs from our Servers to our Logstash Server, we need to create an SSL certificate and key pair. The certificate is used by the Logstash Forwarder to verify the identity of Logstash Server.

On your **Logstash Server** (we just installed it!) generate the SSL Certificates like this:

Search and copy your OpenSSL configuration file, in this case we can find it on /etc/ssl/openssl.cnf ::

 $ sudo cd /etc/pki/tls/
 $ sudo cp /etc/ssl/openssl.cnf .

Edit openssl.cnf file ::

 $ sudo vi /etc/pki/tls/openssl.cnf

Find the [ v3_ca ] section in the file, and add this line under it (substituting in the Logstash Server's private IP address) ::

 $ subjectAltName = IP: logstash_server_private_ip

Save and exit.

Now generate the SSL certificate and private key in the appropriate locations (/etc/pki/tls/), with the following commands ::

 $ cd /etc/pki/tls
 $ sudo openssl req -config /etc/pki/tls/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt

Finally we have our Logstash certificate and key saved on /etc/pki/tls/certs and /etc/pki/tls/private respectively, we will use them soon.

3.2 Copy SSL Certificate
""""""""""""""""""""""""""""""""""""""""""""""""""""""

On **Logstash Server**, copy the SSL certificate to Client Server(Logstash-Forwarder) (substitute the client server's IP address, and your own login)::

 scp /etc/pki/tls/certs/logstash-forwarder.crt user@server_private_IP:/tmp


3.2 Installing
""""""""""""""""""""""""""""""""""""""""""""""""""""""

You can visit Elasticsearch official website and download DEB or RPM packages directly from there. 

`Logstash Forwarder DEB & RPM packages <https://www.elastic.co/downloads/logstash>`_

In this case we are adding DEB repositories and installing by apt-get, proceed to add Logstash-Forwarder repositories, update and install::

 $ wget -qO - https://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
 $ sudo echo "deb http://packages.elasticsearch.org/logstashforwarder/debian stable main" | sudo tee -a /etc/apt/sources.list
 $ sudo apt-get update && sudo apt-get install logstash-forwarder

Now copy the Logstash server's SSL certificate into the appropriate location (/etc/pki/tls/certs)::

 $ sudo cp /tmp/logstash-forwarder.crt /etc/pki/tls/certs/


3.2 Configuring
""""""""""""""""""""""""""""""""""""""""""""""""""""""

Open Logstash Forwarder configuration file, we need to modify some settings to add our certificate recently generated and the Logstash Server IP::

 $ sudo vi /etc/logstash-forwarder.conf

Under the network section, add the following lines into the file, substituting in your Logstash Server IP address for localhost:5043 and uncomment the line ::

 # A list of downstream servers listening for our messages.
 # logstash-forwarder will pick one at random and only switch if
 # the selected one appears to be dead or unresponsive
 "servers": [ "your_logstash_server_ip:5000" ],

Above thoose lines you will fined the CA configuration, edit with our CA path and uncomment the line ::

 # The path to your trusted ssl CA file. This is used
 # to authenticate your downstream server.
 "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt",

Uncomment timeout option line for performance reasons ::

 # logstash-forwarder will assume the connection or server is bad and
 # will connect to a server chosen at random from the servers list.
 "timeout": 15

Finally set LogstashForwarder to fetch **OSSEC ALERTS FILE**, modify following lines like this ::

 # The list of files configurations
 "files": [
  {
     "paths": [
       "/var/ossec/logs/alerts/alerts.json"
      ],
     "fields": { "type": "ossec-alerts" }
 }


Restart and we are finish to configure Logstash Forwarder ::

  $ sudo service logstash-forwarder restart

4. Elasticsearch
^^^^^^^^^^^^^^^^^^^
4.1 Introduction
""""""""""""""""""""""""""""""""""""""""""""""""""""""

4.2 Installing
""""""""""""""""""""""""""""""""""""""""""""""""""""""

4.3 Basic configuration
""""""""""""""""""""""""""""""""""""""""""""""""""""""

4.4 Extra configuration
""""""""""""""""""""""""""""""""""""""""""""""""""""""

4.5 Wazuh custom templates
""""""""""""""""""""""""""""""""""""""""""""""""""""""

5. Kibana
^^^^^^^^^^^^^^^^^^^

Troubleshooting
-------------------------
.. toctree::
   :maxdepth: 2



