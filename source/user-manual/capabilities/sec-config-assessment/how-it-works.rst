.. Copyright (C) 2019 Wazuh, Inc.

How it works
============

- `State vs alerts`_
- `Available information of scans`_
- `Integrity mechanism`_
- `Starting to work`_

State vs alerts
---------------

Agents have their own local database in the form of a hash table where they store the state of each check: *passed* or *failed*. It allows the agent to send only the differences between each scan, if nothing has changed from the last scan, only the summary will be sent, avoiding network flooding every time a scan ends.

On the manager side, the results for each check are stored in the agent's sqlite database. This allows to find out if the state of a check has changed between scans, avoiding if it has changed then an alert is generated for that check.


Available information of scans
------------------------------

Queriable information of the scans
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Every scan comes along with useful information about it. The JSON format is used to send the agents events to the manager.
There are basically three types of events:

- Check event
- Summary event
- Enabled policies event


Example of a check event:

.. code-block:: json

    {  
        "type":"check",
        "id":1479401170,
        "policy":"System audit for SSH hardening",
        "policy_id":"system_audit_ssh",
        "check":{  
            "id":1507,
            "title":"SSH Hardening - 8: Wrong Grace Time.",
            "description":"The option LoginGraceTime should be set to 30.",
            "rationale":"The option LoginGraceTime specifies how long in seconds after a connection request the server will wait before disconnecting if the user has not successfully logged in. 30 seconds is the recommended time for avoiding open connections without authenticate.",
            "remediation":"Change the LoginGraceTime option value in the sshd_config file.",
            "compliance":{  
                "pci_dss":"2.2.4"
            },
            "rules":[  
                "f:$sshd_file -> !r:^\\s*LoginGraceTime\\s+30\\s*$;"
            ],
            "file":"/etc/ssh/sshd_config",
            "result":"failed"
        }
    }


Example of a summary event:

.. code-block:: json

    {  
        "type":"summary",
        "scan_id":1289362433,
        "name":"System audit for password-related vulnerabilities",
        "policy_id":"system_audit_pw",
        "file":"system_audit_pw.yml",
        "description":"Guidance for establishing a secure configuration for password vulnerabilities.",
        "references":"https://www.cisecurity.org/cis-benchmarks/",
        "passed":2,
        "failed":2,
        "score":50,
        "start_time":1556011249,
        "end_time":1556011249,
        "hash":"9e5250664623114d80d97a434fd78c9b7c52e0d3c92e3f448a4720ab1d40d84f",
        "hash_file":"4fe1308a35e1d062718a8e39f9420a740e95a6f145030a0f7e8b169d2b94654b",
        "force_alert":"1"
    }


Example of an enabled policies event:

.. code-block:: json

    {
        "type":"policies",
        "policies":[
            "cis_debian",
            "system_audit",
            "system_audit_ssh"
        ]
    }

The information of the different types of events are stored on the manager side inside the agent's sqlite database. The database has the following tables:

+------------------------------+-------------------------------------------------------------------------------------------+
| Table                        | Description                                                                               |
+------------------------------+-------------------------------------------------------------------------------------------+
| sca_policy                   | Stores the information about the policy file itselt. Related to the summary event         |
+------------------------------+-------------------------------------------------------------------------------------------+
| sca_scan_info                | Stores the information about the scan. Related to the summary event                       |
+------------------------------+-------------------------------------------------------------------------------------------+
| sca_check                    | Stores the information about the check. Related to the check event                        |
+------------------------------+-------------------------------------------------------------------------------------------+
| sca_check_compliance         | Stores the information about the compliances of a check event. Related to the check event |
+------------------------------+-------------------------------------------------------------------------------------------+
| sca_check_rules              | Stores the information about the rules of a check event. Related to the check event       |
+------------------------------+-------------------------------------------------------------------------------------------+


Check status
^^^^^^^^^^^^

A scan event has two different states, they can be ``passed`` or ``failed``. A ``failed`` status is set when the check requirements aren't met.
Take the following example from the cis file ``cis_debian_linux_rcl.yml``:

.. code-block:: yaml

 # Section 3.1 - Network Parameters (Host Only)
 - id: 5031
   title: "Ensure IP forwarding is disabled"
   description: "The net.ipv4.ip_forward and net.ipv6.conf.all.forwarding flags are used to tell the system whether it can forward packets or not."
   rationale: "Setting the flags to 0 ensures that a system with multiple interfaces (for example, a hard proxy), will never be able to forward packets, and therefore, never serve as a router."
   remediation: "Set the following parameter in /etc/sysctl.conf or a /etc/sysctl.d/* file: net.ipv4.ip_forward = 0, net.ipv6.conf.all.forwarding = 0"
   compliance:
    - cis_csc: "5.1"
    - cis: "3.1.1"
   condition: any
   rules:
     - 'f:/proc/sys/net/ipv4/ip_forward -> 1;'
     - 'f:/proc/sys/net/ipv6/ip_forward -> 1;'

The following event is send:

.. code-block:: json

    {  
        "type":"check",
        "id":618748202,
        "policy":"CIS benchmark for Debian/Linux",
        "policy_id":"cis_debian",
        "check":{  
            "id":5031,
            "title":"Ensure IP forwarding is disabled",
            "description":"The net.ipv4.ip_forward and net.ipv6.conf.all.forwarding flags are used to tell the system whether it can forward packets or not.",
            "rationale":"Setting the flags to 0 ensures that a system with multiple interfaces (for example, a hard proxy), will never be able to forward packets, and therefore, never serve as a router.",
            "remediation":"Set the following parameter in /etc/sysctl.conf or a /etc/sysctl.d/* file: net.ipv4.ip_forward = 0, net.ipv6.conf.all.forwarding = 0",
            "compliance":{  
                "cis_csc":"5.1",
                "cis":"3.1.1"
            },
            "rules":[  
                "f:/proc/sys/net/ipv4/ip_forward -> 1;",
                "f:/proc/sys/net/ipv6/ip_forward -> 1;"
            ],
            "file":"/proc/sys/net/ipv4/ip_forward,/proc/sys/net/ipv6/ip_forward",
            "result":"passed"
        }
    }

The *result* is ``passed`` because the ``rules`` are looking for a ``1`` inside the ``/proc/sys/net/ipv4/ip_forward`` file or the ``/proc/sys/net/ipv6/ip_forward``. 
As we have the value ``0`` in both files, the result is marked as ``passed``.

.. note::
  A *check* can be marked as *not applicable* in the case that an error happens getting the result.
  In this case, the field *result* doesn't appear and the check returns two fields: *status* and *reason*.

  
Enabled policies
^^^^^^^^^^^^^^^^

Each agent will send the policies it has enabled, so the manager can compare them with the agent database and erase the disabled policies (if any).

.. code-block:: json

    {
        "type":"policies",
        "policies":[
            "cis_debian",
            "system_audit",
            "system_audit_ssh"
        ]
    }

The ``sca_policy`` table will be queried, comparing the existing policies with the policies from the event above.


Integrity mechanism
-------------------

To maintain the correct correlation between the agent state for each check and the manager's database for that agent, an integrity mechanism has been developed.

Integrity of the result
^^^^^^^^^^^^^^^^^^^^^^^

Let's look at how it works with an example.

On the agent side we have the following hash table:

+------------------------------+----------------+
| Check ID                     | State          |
+------------------------------+----------------+
| 1000                         | passed         |
+------------------------------+----------------+
| 1001                         | failed         |
+------------------------------+----------------+
| 1002                         | failed         |
+------------------------------+----------------+
| 1003                         | passed         |
+------------------------------+----------------+

It will send an SHA256 hash inside the summary event with the result of the concatenation ``passed,failed,failed,passed`` being it ``1642AB1DC478052AC3556B5E700CD82ADB69728008301882B9CBEE0696FF2C84``.

On the ``manager`` side, let's asume the database is as follows:

+------------------------------+----------------+
| Check ID                     | State          |
+------------------------------+----------------+
| 1000                         | passed         |
+------------------------------+----------------+
| 1001                         | failed         |
+------------------------------+----------------+
| 1003                         | passed         |
+------------------------------+----------------+

The ID 1002 is missing, so the concatenation ``passed,failed,passed`` produces the SHA256 hash ``B43037CA28D95A69B6F9E03FCD826D2B253A6BB1B6AD28C4AE57A3A766ACE610``.
As the SHA256 of the agent ``1642AB1DC478052AC3556B5E700CD82ADB69728008301882B9CBEE0696FF2C84`` and the SHA256 of the manager ``B43037CA28D95A69B6F9E03FCD826D2B253A6BB1B6AD28C4AE57A3A766ACE610`` do not match, the manager will request a full database dump to the agent.

Integrity of the files
^^^^^^^^^^^^^^^^^^^^^^

Now, let's see what happens when a policy file is changed but his information is stored in the database:

The SHA256 of the policy file ``system_audit_ssh`` is: ``BFA7204C70C5C0DA65E351BDAC27F56FE3074DF17AEA27475BEE695770D2C951``
This SHA256 is stored in the table `sca_policy` of the agent database.

Let's change the rule that search for the maximum authentication tries to check if the maximum is 3 instead of 4:
``- 'f:$sshd_file -> !r:^\s*MaxAuthTries\s+4\s*$;'`` -> ``- 'f:$sshd_file -> !r:^\s*MaxAuthTries\s+3\s*$;'``

Now the SHA256 of the file is: ``339DD252D55574B265DD3384DBC33EBC801EED7ECA5870CD70FECCC868E73725``

In the next SCA scan the manager will compare both SHA256. If they are different, as it is in our case, the manager will wipe the information referred to the policy that changed (``system_audit_ssh``) and the manager will request a policy information dump to the agent.

Starting to work
----------------

During a policy file scan it follows the steps described below:

- Load and parse the policy file
- Check if the requirements are meet (if any)
- Execute each rule defined for each check
- Compare the result for every check with the internal database (passed or failed)
- If it has changed, send an alert to the manager and update the database
- Send the policy summary
- Send the enabled policies
