.. Copyright (C) 2019 Wazuh, Inc.

.. _password-authorization-registration:

Registration service with password authorization
================================================

This registration method is similar to the :ref:`simple registration service <simple-registration-service>`, except that it allows additional protection of the Wazuh manager from unauthorized registrations by using a password.

Wazuh manager
^^^^^^^^^^^^^

1. To enable password authorization amend the Wazuh manager's ``/var/ossec/etc/ossec.conf`` configuration file as shown below:

  .. code-block:: xml

    <auth>
      ...
      <use_password>yes</use_password>
      ...
    </auth>

2. Choose custom password or let the registration service generate one.

  .. tabs::

   .. group-tab:: Using a custom password

    Create the ``/var/ossec/etc/authd.pass`` file and save the custom password in it.

    In the command below, replace ``<custom_pasword>`` with your chosen password:

    .. code-block:: console

      # echo "<custom_password>" > /var/ossec/etc/authd.pass

   .. group-tab:: Using a random password

    If no password is specified in the ``/var/ossec/etc/authd.pass`` file, the registration service will create a random password. The password can be found in ``/var/ossec/logs/ossec.log`` by executing the following command:

    .. code-block:: console

      # grep "Random password" /var/ossec/logs/ossec.log

    .. code-block:: none
             :class: output

             2019/04/25 15:09:50 ossec-authd: INFO: Accepting connections on port 1515. Random password chosen for agent authentication: 3027022fa85bb4c697dc0ed8274a4554


3. Restart the Wazuh manager for the changes to take effect:

 .. include:: ../../_templates/registrations/common/restart_manager.rst

Wazuh agents
^^^^^^^^^^^^

Choose the tab corresponding to the Wazuh agent host operating system:

.. tabs::

 .. group-tab:: Linux/Unix host

   Open a terminal in the Linux/Unix Wazuh agent's host as a ``root`` user.

   1. Register the Wazuh agent using the password. It can be stored in a file or provided as a command-line argument:

     .. tabs::

      .. group-tab:: Using a stored password

       Write the password on ``/var/ossec/etc/authd.pass`` file and run the ``agent-auth`` utility using the Wazuh manager’s IP address:

       .. code-block:: console

        # echo "<custom_password>" > /var/ossec/etc/authd.pass
        # /var/ossec/bin/agent-auth -m <manager_IP>

       .. include:: ../../_templates/registrations/common/set_agent_name.rst



      .. group-tab:: Using a password as a command-line argument

       Run the ``agent-auth`` utility providing the Wazuh manager’s IP address together with the password followed by the ``-P`` flag:

       .. code-block:: console

        # /var/ossec/bin/agent-auth -m <manager_IP> -P "<custom_password>"

       .. include:: ../../_templates/registrations/common/set_agent_name.rst



   2. To enable the communication with the Wazuh manager, edit the Wazuh agent's ``/var/ossec/etc/ossec.conf`` configuration file:

    .. include:: ../../_templates/registrations/common/client_server_section.rst

   3. Restart the Wazuh agent:

    .. include:: ../../_templates/registrations/linux/restart_agent.rst

   The Wazuh agent registration can be adjusted by using different :ref:`agent-auth` options.



 .. group-tab:: Windows host

   Open a Powershell or CMD session in the Wazuh agent's host as an ``Administrator``.

   .. include:: ../../_templates/registrations/windows/installation_directory.rst

   1. Register the Wazuh agent using the password. It can be stored in a file or provided as a command-line argument:

     .. tabs::

      .. group-tab:: Using a stored password

       Write the password on ``C:\Program Files (x86)\ossec-agent\authd.pass`` file and run the ``agent-auth`` utility using the Wazuh manager’s IP address:

       .. code-block:: none

        # echo <custom_password> > "C:\Program Files (x86)\ossec-agent\authd.pass"
        # C:\Program Files (x86)\ossec-agent\agent-auth.exe -m <manager_IP>

       .. include:: ../../_templates/registrations/common/set_agent_name.rst

       The Wazuh agent assumes that the input file is in ``UTF-8`` encoding, without ``byte-order mark (BOM)``. If the file is created in an incorrect encoding it can be changed by opening the ``authd.pass`` file in a Notepad and Save As ``ANSI`` encoding.



      .. group-tab:: Using a password as a command-line argument

       Run the ``agent-auth`` utility, provide the Wazuh manager’s IP address together with the password following the ``-P`` flag:

       .. code-block:: none

         # C:\Program Files (x86)\ossec-agent\agent-auth.exe -m <manager_IP> -P "<custom_password>"

       .. include:: ../../_templates/registrations/common/set_agent_name.rst



   2. To enable the communication with the Wazuh manager, edit the Wazuh agent's ``C:\Program Files (x86)\ossec-agent\ossec.conf`` configuration file:

    .. include:: ../../_templates/registrations/common/client_server_section.rst

   3. Restart the Wazuh agent:

    .. include:: ../../_templates/registrations/windows/restart_agent.rst

   The Wazuh agent registration can be adjusted by using different :ref:`agent-auth` options.



 .. group-tab:: MacOS X host

  Open a terminal in the Linux/Unix Wazuh agent's host as a ``root`` user.

  1. Register the Wazuh agent using the password. It can be stored in a file or provided as a command-line argument:

    .. tabs::

     .. group-tab:: Using a stored password

      Write the password on ``/Library/Ossec/etc/authd.pass`` file and run the ``agent-auth`` utility using the Wazuh manager’s IP address:

      .. code-block:: console

         # echo "<custom_password>" > /Library/Ossec/etc/authd.pass
         # /Library/Ossec/bin/agent-auth -m <manager_IP>

      .. include:: ../../_templates/registrations/common/set_agent_name.rst



     .. group-tab:: Using a password as a command-line argument

      Run the ``agent-auth`` utility, provide the Wazuh manager’s IP address together with the password following the ``-P`` flag:

      .. code-block:: console

        # /Library/Ossec/bin/agent-auth -m <manager_IP> -P "<custom_password>"

      .. include:: ../../_templates/registrations/common/set_agent_name.rst


  2. To enable the communication with the Wazuh manager, edit the Wazuh agent's ``/Library/Ossec/etc/ossec.conf`` configuration file:

   .. include:: ../../_templates/registrations/common/client_server_section.rst

  3. Restart the Wazuh agent:

   .. include:: ../../_templates/registrations/macosx/restart_agent.rst

  The Wazuh agent registration can be adjusted by using different :ref:`agent-auth` options.
