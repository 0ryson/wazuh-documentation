.. Copyright (C) 2019 Wazuh, Inc.

.. _automated_deployment_variables:

Automated deployment variables
==============================

Traditionally, if we want to have a new Wazuh Agent reporting to a Wazuh Manager we had to follow three steps: installation, registration, and configuration.  We designed an automated deployment embedded in the installation stage. The three steps previously mentioned became only one command line.  By using declared variables, the automated deployment process will take those variables and it will make convenient changes in ossec.conf and in the registration run. 

In the table below, you'll find all the options available for the automated deployment included in the installation. 


+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
| Option                | Description                                                                                                                  |
+=======================+==============================================================================================================================+
|   WAZUH_MANAGER_IP    |  Specifies the managers IP address or hostname. You can add multiple values by commas.                                       |
|                       |                                                                                                                              |
|                       |  See `address <../../user-manual/reference/ossec-conf/client.html#address>`_.                                                |
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_MANAGER_PORT  |  Specifies the managers connection port.                                                                                     |
|                       |                                                                                                                              |
|                       |  See `server-port <../../user-manual/reference/ossec-conf/client.html#server-port>`_.                                        |
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_PROTOCOL      |  Sets the communication protocol between the manager and the agent. Accepts UDP and TCP. Default is UDP.                     |
|                       |                                                                                                                              |
|                       |  See `server-protocol <../../user-manual/reference/ossec-conf/client.html#server-protocol>`_.                                |
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_AUTHD_SERVER  |  Specifies the Wazuh authentication server.                                                                                  |
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_AUTHD_PORT    |  Specifies the port used by the Wazuh authentication server.                                                                 |
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_PASSWORD      |  Sets the Wazuh authentication server.                                                                                       |
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |    
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_NOTIFY_TIME   |  Sets the time between agent checks for manager connection.                                                                  |
|                       |                                                                                                                              |    
|                       |  See `notify-time <../../user-manual/reference/ossec-conf/client.html#notify-time>`_.                                        |    
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_TIME_RECONNECT|  Sets the time in seconds until a reconnection attempt if the connection between agent and manager is lost.                  |
|                       |                                                                                                                              |
|                       |  See `time-reconnect <../../user-manual/reference/ossec-conf/client.html#time-reconnect>`_.                                  |
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_CERTIFICATE   |  Host SSL validation need of Certificate of Authority. This option specifies the CA path.                                    |
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |   
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_PEM           |  The SSL agent verification needs a CA signed certificate and the respective key. This option specifies the certificate path.|
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |    
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_KEY           |  Specifies the key path completing the required variables with WAZUH_PEM for the SSL agent verification process.             |
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |    
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_AGENT_NAME    |  Designates the agent's name. By default it will be the computer name.                                                       |
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |    
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+
|   WAZUH_GROUP         |  Assigns the installed agent to a previously created group.                                                                  |
|                       |                                                                                                                              |
|                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_.                                              |    
+-----------------------+------------------------------------------------------------------------------------------------------------------------------+


Examples:

* Registration with password (using `apt-get`):

.. code-block:: console

     # WAZUH_MANAGER_IP="192.168.1.2" WAZUH_PASSWORD="TopSecret" \
          WAZUH_AGENT_NAME="ubuntu18" apt-get install wazuh-agent

* Registration with password and assigning a group (using `yum`):

.. code-block:: console

     # WAZUH_MANAGER_IP="192.168.1.2" WAZUH_AUTHD_SERVER="192.168.1.2" WAZUH_PASSWORD="TopSecret" \
          WAZUH_GROUP="my-group" yum install wazuh-agent

* Registration with relative path to CA. It will be searched at your Wazuh installation folder (using `rpm -i` in AIX):

.. code-block:: console

     # WAZUH_MANAGER_IP="192.168.1.2" WAZUH_AUTHD_SERVER="192.168.1.2" WAZUH_AGENT_NAME="ubuntu18" \
     WAZUH_CERTIFICATE="rootCA.pem" rpm -i wazuh-agent-3.9.0-1.aix.ppc.rpm

* Registration with protocol (using `apt-get`):

.. code-block:: console

     # WAZUH_MANAGER_IP="192.168.1.2" WAZUH_AUTHD_SERVER="192.168.1.2" WAZUH_AGENT_NAME="ubuntu18" \
          WAZUH_PROTOCOL="tcp" apt-get install wazuh-agent

* Registration and adding multiple address (using `yum`):

.. code-block:: console

     # WAZUH_MANAGER_IP="192.168.1.2,192.168.1.3" WAZUH_AUTHD_SERVER="192.168.1.2" \
          WAZUH_AGENT_NAME="ubuntu18" apt-get install wazuh-agent

* Absolute paths to CA, certificate or key that contain spaces can be written as shown below (in MacOS):

.. code-block:: console

     # launchctl setenv WAZUH_MANAGER_IP "192.168.1.2" WAZUH_AUTHD_SERVER "192.168.1.2" WAZUH_KEY "/var/ossec/etc/sslagent.key" \
          WAZUH_PEM "/var/ossec/etc/sslagent.cert" && installer -pkg wazuh-agent-3.9.0-1.pkg -target /

.. note:: To verify agents via SSL, it's necessary to use both KEY and PEM options. See the :ref:`verify hosts with SSL <verify-hosts>` section.
