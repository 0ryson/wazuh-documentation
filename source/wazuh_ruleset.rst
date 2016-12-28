.. _wazuh_ruleset:

Wazuh Ruleset
=============

This documentation explains how to install, update, and contribute to OSSEC HIDS Ruleset mantained by Wazuh. These rules are used by the system to detect attacks, intrusions, software misuse, configuration problems, application errors, malware, rootkits, system anomalies or security policy violations. OSSEC provides an out-of-the-box set of rules that we update by modifying them or including new ones, in order to increase OSSEC detection capabilities.


GitHub repository
------------------

In the ruleset repository you will find:

* **New rules, decoders and rootchecks**
   OSSEC default number of rules and decoders is limited. For this reason, we centralize, test and maintain decoders and rules submitted by Open Source contributors. As well, we create new rules and rootchecks periodically that are added to this repository so they can be used by the users community. Some examples are the new rules for Netscaler and Puppet.
   We update and maintain out-of-the-box rules provided by OSSEC, both to eliminate false positives or to increase their accuracy. In addition, we map those with PCI-DSS compliance controls, making it easy to identify when an alert is related to a compliance requirement.

* **Tools**
   We provide some usefull tools for testing.


Resources
^^^^^^^^^

* Visit our repository to view the rules in detail at `Github Wazuh Ruleset <https://github.com/wazuh/wazuh-ruleset>`_
* Find a complete description of the available rules: `Wazuh Ruleset Summary <http://www.wazuh.com/resources/OSSEC_Ruleset.pdf>`_

Rule and Rootcheck example
^^^^^^^^^^^^^^^^^^^^^^^^^^

Log analysis rule for Netscaler with PCI DSS compliance mapping:
::

    <rule id="80102" level="10" frequency="6">
        <if_matched_sid>80101</if_matched_sid>
        <same_source_ip />
        <description>Netscaler: Multiple AAA failed to login the user</description>
        <group>authentication_failures,netscaler-aaa,pci_dss_10.2.4,pci_dss_10.2.5,pci_dss_11.4,</group>
    </rule>

Rootcheck rule for SSH Server with mapping to CIS security benchmark and PCI DSS compliance:
::

   [CIS - Debian Linux - 2.3 - SSH Configuration - Empty passwords permitted {CIS: 2.3 Debian Linux} {PCI_DSS: 4.1}] [any] [http://www.ossec.net/wiki/index.php/CIS_DebianLinux]
   f:/etc/ssh/sshd_config -> !r:^# && r:^PermitEmptyPasswords\.+yes;


Directory layout
------------------

The ruleset folder structure is divided in two parts following the next schema:
::

   /var/ossec/
           ├─ etc/
           │   ├─ decoders
           │   └─ rules
           └─ ruleset/
                   ├─ decoders
                   └─ rules

Folders
^^^^^^^^^^^^^^^
Inside the ``etc/`` folder we will find the ``local_decoder.xml`` and ``local_rules.xml`` files inside their corresponding folders.
Use these folders to keep you personal decoders and rules.

Inside the ``ruleset/`` folder we will find all the common rules and decoders files. This folder will be overwritten or modified in the Wazuh update process, so please do not use this folder for personal files and use the ``etc/`` folder instead.

You will be able to include or exclude decoders and rules, and also add new folders.


Update ruleset
----------------

Run ``update_ruleset.py`` script to update Wazuh Ruleset with no need to manually change OSSEC internal files.


Usage examples
^^^^^^^^^^^^^^

Update Decoders, Rules and Rootchecks: ::

   $ /var/ossec/bin/update_ruleset.py

All script options: ::

  Restart:
    -r, --restart       Restart OSSEC when required.
    -R, --no-restart    Do not restart OSSEC when required.

  Backups:
    -b, --backups       Restore last backup.

  Additional Params:
    -f, --force-update  Force to update the ruleset. By default, only it is updated the new/changed decoders/rules/rootchecks.
    -o, --ossec-path    Set OSSEC path. Default: '/var/ossec'
    -s, --source        Select ruleset source path (instead of download it).
    -j, --json          JSON output. It should be used with '-s' or '-S' argument.
    -d, --debug         Debug mode.


Configure weekly updates
^^^^^^^^^^^^^^^^^^^^^^^^

Run ``update_ruleset.py`` weekly and keep your Wazuh Ruleset installation up to date by adding a crontab job to your system.

Run ``sudo crontab -e`` and, at the end of the file, add the following line ::

  @weekly root cd /var/ossec/bin && ./update_ruleset.py -r



Custom rules and decoders
----------------------------
ToDo

Adding new decoders and rules
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You can add new rules and decoders to your ruleset in order to improve the monitoring process.

We are going to describe these procedures using an easy example. We have the next log from a program called ``example``:
::

   Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100'

First, we need to decode this information. We create the new decoder code into the file ``/var/ossec/etc/decoders/local_decoder.xml``:
::

  <decoder name="example">
    <program_name>^example</program_name>
  </decoder>

  <decoder name="example">
    <parent>example</parent>
    <regex>User '(\w+)' logged from '(\d+.\d+.\d+.\d+)'</regex>
    <order>user, srcip</order>
  </decoder>

.. note::
   For small changes we will use the file ``local_decoder.xml``, in other case we will create a new decoder file.

Now, we will create the next rule into the file ``/var/ossec/etc/rules/local_rules.xml``:
::

  <rule id="100010" level="0">
    <program_name>example</program_name>
    <description>User logged</description>
  </rule>

.. note::
   For small changes we will use the file ``local_rules.xml``, in other case we will create a new rule file.

We can check if it works using ``/var/ossec/bin/ossec-logtest``:
::

  **Phase 1: Completed pre-decoding.
       full event: 'Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100''
       hostname: 'MyHost'
       program_name: 'example'
       log: 'User 'admin' logged from '192.168.1.100''

  **Phase 2: Completed decoding.
       decoder: 'example'
       dstuser: 'admin'
       srcip: '192.168.1.100'

  **Phase 3: Completed filtering (rules).
       Rule id: '100010'
       Level: '0'
       Description: 'User logged'



Changing existing rule
^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to modify the installed rules.

.. warning::
    All changes in any rule file inside the ``/var/ossec/ruleset/rules`` folder will be lost in the update process. Use the next procedure to keep your changes.

For example, if we want to change the level value of the SSH rule ``5710`` from 5 to 10, we will follow the next steps:

1. Open the rule file ``/var/ossec/ruleset/rules/0095-sshd_rules.xml``.

2. Search and copy the following XML code:

::

  <rule id="5710" level="5">
    <if_sid>5700</if_sid>
    <match>illegal user|invalid user</match>
    <description>sshd: Attempt to login using a non-existent user</description>
    <group>invalid_login,authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5,pci_dss_10.6.1,</group>
  </rule>

3. Paste the code into the file ``/var/ossec/etc/rules/local_rules.xml``, modify the level value and add ``overwrite="yes"`` to indicate that this rule will be overwritten:

::

  <rule id="5710" level="10" overwrite="yes">
    <if_sid>5700</if_sid>
    <match>illegal user|invalid user</match>
    <description>sshd: Attempt to login using a non-existent user</description>
    <group>invalid_login,authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5,pci_dss_10.6.1,</group>
  </rule>


Changing existing decoder
^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to modify the installed decoders.

.. warning::
    All changes in any decoder file inside the ``/var/ossec/ruleset/decoders`` folder will be lost in the update process. Use the next procedure to keep your changes.

Unfortunately, there is any automatic procedure to overwrite decoders like the procedure described above for rules. However, we can perform changes in any decoder file following the next steps:

1. Copy the decoder file from the default folder ``/var/ossec/ruleset/decoders`` to the user folder ``/var/ossec/etc/decoders``.

2. Exclude the original decoder file from the OSSEC loading list. To do this, use the tag ``<exclude>`` in the ``ossec.conf`` file.

3. Perform the changes in the file placed in the user folder ``/var/ossec/etc/decoders``.



Contribute to the ruleset
-------------------------

If you have created new rules, decoders or rootchecks and you would like to contribute to our repository, please fork our `Github repository <https://github.com/wazuh/wazuh-ruleset>`_ and submit a pull request.

If you are not familiar with Github, you can also share them through our `users mailing list <https://groups.google.com/d/forum/wazuh>`_, to which you can subscribe by sending an email to ``wazuh+subscribe@googlegroups.com``. As well do not hesitate to request new rules or rootchecks that you would like to see running in OSSEC and our team will do our best to make it happen.

.. note:: In our repository you will find that most of the rules contain one or more groups called pci_dss_X. This is the PCI DSS control related to the rule. We have produced a document that can help you tag each rule with its corresponding PCI requirement: http://www.wazuh.com/resources/PCI_Tagging.pdf

What's next
-----------

Once you have your ruleset up to date we encourage you to move forward and try out ELK integration or the API RESTful, check them on:


* :ref:`ELK Stack integration guide <ossec_elk>`
* :ref:`Wazuh RESTful API installation Guide <wazuh_api>`
