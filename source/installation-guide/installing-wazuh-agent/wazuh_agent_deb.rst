.. Copyright (C) 2019 Wazuh, Inc.

.. _wazuh_agent_deb:

Install Wazuh agent with DEB packages
=====================================

The DEB packages are suitable for Debian, Ubuntu, and other Debian-based systems.

.. note:: Many of the commands described below need to be executed with root user privileges.

Adding the Wazuh repository
---------------------------

The first step to installing the Wazuh agent is to add the Wazuh repository to your server. Alternatively, if you prefer to download the wazuh-agent package directly, you can find it :ref:`here <packages>`.

1. To perform this procedure, the ``curl``, ``apt-transport-https`` and ``lsb-release`` packages must be installed on your system. If they are not already present, install them using the commands below:

  .. code-block:: console

    # apt-get install curl apt-transport-https lsb-release

2. Install the Wazuh repository GPG key:

  .. code-block:: console

    # curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -

3. Add the repository:

  .. code-block:: console

    # echo "deb https://packages.wazuh.com/3.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list

4. Update the package information:

  .. code-block:: console

    # apt-get update

Installing Wazuh agent
----------------------

1. On your terminal, install the Wazuh agent:

  .. code-block:: console

    # apt-get install wazuh-agent

  You can automate the agent registration with authd using the following environment variables:

+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
     | Option                | Description                                                                                                                  |
     +=======================+==============================================================================================================================+
     |   WAZUH_MANAGER_IP    |  Specifies the managers IP address or hostname. You can add multiple values by commas.                                       |
     |                       |                                                                                                                              |
     |                       |  See `address <../../user-manual/reference/ossec-conf/client.html#address>`_                                                 |
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_SERVER_PORT   |  Specifies the managers connection port.                                                                                     |
     |                       |                                                                                                                              |
     |                       |  See `server-port <../../user-manual/reference/ossec-conf/client.html#server-port>`_                                         |
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_PROTOCOL      |  Sets the communication protocol between the manager and the agent. Accepts UDP and TCP. Default is UDP.                     |
     |                       |                                                                                                                              |
     |                       |  See `server-protocol <../../user-manual/reference/ossec-conf/client.html#server-protocol>`_                                 |
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_AUTHD_SERVER  |  Specifies the Wazuh authentication server.                                                                                  |
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_AUTHD_PORT    |  Specifies the port used by the Wazuh authentication server.                                                                 |
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_PASSWORD      |  Sets the Wazuh authentication server.                                                                                       |
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |    
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_NOTIFY_TIME   |  Sets the time between agent checks for manager connection.                                                                  |
     |                       |                                                                                                                              |    
     |                       |  See `notify-time <../../user-manual/reference/ossec-conf/client.html#notify-time>`_                                         |    
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_TIME_RECONNECT|  Sets the time in seconds until a reconnection attempt if the connection between agent and manager is lost.                  |
     |                       |                                                                                                                              |
     |                       |  See `time-reconnect <../../user-manual/reference/ossec-conf/client.html#time-reconnect>`_                                   |
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_CERTIFICATE   |  Host SSL validation need of Certificate of Authority. This option specifies the CA path.                                    |
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |   
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_PEM           |  The SSL agent verification needs a CA signed certificate and the respective key. This option specifies the certificate path.|
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |    
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_KEY           |  Specifies the key path completing the required variables with WAZUH_PEM for the SSL agent verification process.             |
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |    
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_AGENT_NAME    |  Designates the agent's name. By default it will be the computer name.                                                       |
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |    
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+
     |   WAZUH_GROUP         |  Assigns the installed agent to a previously created group.                                                                  |
     |                       |                                                                                                                              |
     |                       |  See `agent-auth options <../../user-manual/reference/tools/agent-auth.html>`_                                               |    
     +-----------------------+------------------------------------------------------------------------------------------------------------------------------+

  Below there are some examples to install and register a Debian-based linux agent.

  Registration with password:

  .. code-block:: console

           # WAZUH_MANAGER_IP="192.168.1.1" WAZUH_AUTHD_SERVER="192.168.1.1" WAZUH_PASSWORD="TopSecret" \
                WAZUH_AGENT_NAME="ubuntu18" apt-get install wazuh-agent

  Registration with password and assigning a group:

  .. code-block:: console

           # WAZUH_MANAGER_IP="192.168.1.1" WAZUH_AUTHD_SERVER="192.168.1.1" WAZUH_PASSWORD="TopSecret" \
                WAZUH_GROUP="my-group" apt-get install wazuh-agent

  Registration with relative path to CA. It will be searched at your Wazuh installation folder:

  .. code-block:: console

           # WAZUH_MANAGER_IP="192.168.1.1" WAZUH_AUTHD_SERVER="192.168.1.1" WAZUH_AGENT_NAME="ubuntu18" \
              WAZUH_CERTIFICATE="rootCA.pem" apt-get install wazuh-agent

  Absolute paths to CA, certificate or key that contain spaces can be written as shown below:

  .. code-block:: console

           # WAZUH_MANAGER_IP="192.168.1.1" WAZUH_AUTHD_SERVER="192.168.1.1" WAZUH_KEY="/var/ossec/etc/sslagent.key" \
                WAZUH_PEM="/var/ossec/etc/sslagent.cert" apt-get install wazuh-agent

  .. note::
      To verify agents via SSL, it's necessary to use both KEY and PEM options. See the :ref:`verify hosts with SSL <verify-hosts>` section.

  Registration with protocol:

  .. code-block:: console

           # WAZUH_MANAGER_IP="192.168.1.1" WAZUH_AUTHD_SERVER="192.168.1.1" WAZUH_AGENT_NAME="ubuntu18" \
                WAZUH_PROTOCOL="tcp" apt-get install wazuh-agent

  Registration and adding multiple address:

  .. code-block:: console

           # WAZUH_MANAGER_IP="192.168.1.1,192.168.1.2" WAZUH_AUTHD_SERVER="192.168.1.1" \
                WAZUH_AGENT_NAME="ubuntu18" apt-get install wazuh-agent

2. (Optional) Disable the Wazuh updates:

  It is recommended that the Wazuh repository be disabled in order to prevent accidental upgrades. To do this, use the following command:

  .. code-block:: console

    # sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/wazuh.list
    # apt-get update

  Alternately, you can set the package state to ``hold``, which will stop updates (although you can still upgrade it manually using ``apt-get install``).

  .. code-block:: console

    # echo "wazuh-agent hold" | sudo dpkg --set-selections

.. note:: Now that the agent is installed, the next step is to register and configure it to communicate with the manager (if you did not use the automatic registering method). For more information about this process, please visit the :doc:`user manual<../../user-manual/agents/registering/index>`.
