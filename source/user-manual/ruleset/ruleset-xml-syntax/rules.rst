.. Copyright (C) 2018 Wazuh, Inc.

.. _rules_syntax:

Rules Syntax
============

After the decoding process, the analysis engine iterates over all the ruleset looking for occurrences that match a rule in order to determine if the log would generate an alert.

Available options
-----------------

- `rule`_
- `match`_
- `regex`_
- `decoded_as`_
- `category`_
- `field`_
- `srcip`_
- `dstip`_
- `extra_data`_
- `user`_
- `program_name`_
- `hostname`_
- `time`_
- `weekday`_
- `id`_
- `url`_
- `action`_
- `if_sid`_
- `if_group`_
- `if_level`_
- `if_matched_sid`_
- `if_matched_group`_
- `same_id`_
- `same_source_ip`_
- `same_src_port`_
- `same_dst_port`_
- `same_field`_
- `same_location`_
- `same_user`_
- `different_url`_
- `different_srcgeoip`_
- `description`_
- `list`_
- `info`_
- `options`_
- `check_diff`_
- `group`_
- `status`_
- `location`_
- `var`_

rule
^^^^

+---------------+----------------+----------------------------------------------------------------------------------------+
| **level**     | Definition     | Specifies the level of the rule. Alerts and responses use this value.                  |
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | 0 to 16                                                                                |
+---------------+----------------+----------------------------------------------------------------------------------------+
| **id**        | Definition     | Specifies the ID of the rule.                                                          |
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | Any number from 1 to 999999                                                            |
+---------------+----------------+----------------------------------------------------------------------------------------+
| **maxsize**   | Definition     | Specifies the maximum size of the event.                                               |
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | Any number from 1 to 9999                                                              |
+---------------+----------------+----------------------------------------------------------------------------------------+
| **frequency** | Definition     | Number of times the rule must have matched before firing.                              |
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | Any number from 2 to 9999                                                              |
+---------------+----------------+----------------------------------------------------------------------------------------+
| **timeframe** | Definition     | The timeframe in seconds. This option is intended to be used with the frequency option.|
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | Any number from 1 to 99999                                                             |
+---------------+----------------+----------------------------------------------------------------------------------------+
| **ignore**    | Definition     | The time (in seconds) to ignore this rule after firing it (to avoid floods).           |
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | Any number from 1 to 999999                                                            |
+---------------+----------------+----------------------------------------------------------------------------------------+
| **overwrite** | Definition     | Used to supersede an OSSEC rule with local changes.                                    |
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | yes, no                                                                                |
+---------------+----------------+----------------------------------------------------------------------------------------+
| **noalert**   | Definition     | Not trigger any alert if the rule matches.                                             |
+               +----------------+----------------------------------------------------------------------------------------+
|               | Allowed values | Attribute with no value                                                                |
+---------------+----------------+----------------------------------------------------------------------------------------+

Multiple of this field can be added to the same rule as follows:

  .. code-block:: xml

    <rule id="100010" level="1" ignore="10">
      (Rule content)
    </rule>


match
^^^^^
Any string to match against the log event.

+--------------------+-----------------------------------------------------------------+
| **Default Value**  | n/a                                                             |
+--------------------+-----------------------------------------------------------------+
| **Allowed values** | Any `sregex expression <regex.html#os-match-or-sregex-syntax>`_ |
+--------------------+-----------------------------------------------------------------+

The *analysis engine* will search in the log the content of this label to check if a rule should trigger.

As an example, here is a custom rule that triggers when a log that contains the word **Error** is catched:

  .. code-block:: xml

    <rule id="100005" level="4">
      <match>Error</match>
      <description> Rule that matches errors. </description>
    </rule>

regex
^^^^^
Any regex to match against the log event.

+--------------------+---------------------------------------------------------------+
| **Default Value**  | n/a                                                           |
+--------------------+---------------------------------------------------------------+
| **Allowed values** | Any `regex expression <regex.html#os-regex-or-regex-syntax>`_ |
+--------------------+---------------------------------------------------------------+

Example:

    An example is this regex that matches any IP address:

  .. code-block:: xml

    <regex>(\d+.\d+.\d+.\d+)</regex>


decoded_as
^^^^^^^^^^

Specifies the decoder name which generated the decoded event log.

+--------------------+------------------+
| **Default Value**  | n/a              |
+--------------------+------------------+
| **Allowed values** | Any decoder name |
+--------------------+------------------+

As a example, if we have a decoder named *mydecoder*:

  .. code-block:: xml

    <decoded_as> mydecoder </decoded_as>

category
^^^^^^^^

Selects in which rule decoding category the rule should be included: ids, syslog, firewall, web-log, squid or windows.

+--------------------+--------------------+
| **Default Value**  | n/a                |
+--------------------+--------------------+
| **Allowed values** | Any valid category |
+--------------------+--------------------+

field
^^^^^

Any `sregex <regex.html#os-match-or-sregex-syntax>`_ to be compared to a dynamic field extracted by the decoder.

+----------+-----------------------------------------------------------+
| **name** | Specifies the name of the field extracted by the decoder. |
+----------+-----------------------------------------------------------+

Here is an example of this option, which rule will only trigger when the *level* field includes a "10", it does not matter if the other fields of the log include it.

  .. code-block:: xml

    <field name="level"> 10 </field>

srcip
^^^^^

Any IP address or CIDR block to be compared to an IP decoded as srcip. Use "!" to negate it.

+--------------------+-----------+
| **Default Value**  | n/a       |
+--------------------+-----------+
| **Allowed values** | Any srcip |
+--------------------+-----------+

dstip
^^^^^

Any IP address or CIDR block to be compared to an IP decoded as dstip. Use "!" to negate it.

+--------------------+-----------+
| **Default Value**  | n/a       |
+--------------------+-----------+
| **Allowed values** | Any dstip |
+--------------------+-----------+

extra_data
^^^^^^^^^^

Any string that is decoded into the extra_data field.

+--------------------+-------------+
| **Default Value**  | n/a         |
+--------------------+-------------+
| **Allowed values** | Any string. |
+--------------------+-------------+

user
^^^^

Any username (decoded as the username).

+--------------------+------------------------------------------------------------------+
| **Default Value**  | n/a                                                              |
+--------------------+------------------------------------------------------------------+
| **Allowed values** | Any `sregex expression <regex.html#os-match-or-sregex-syntax>`_  |
+--------------------+------------------------------------------------------------------+

program_name
^^^^^^^^^^^^

Program name is decoded from syslog process name.

+--------------------+------------------------------------------------------------------+
| **Default Value**  | n/a                                                              |
+--------------------+------------------------------------------------------------------+
| **Allowed values** | Any `sregex expression <regex.html#os-match-or-sregex-syntax>`_  |
+--------------------+------------------------------------------------------------------+

hostname
^^^^^^^^

Any hostname (decoded as the syslog hostname) or log file.

+--------------------+------------------------------------------------------------------+
| **Default Value**  | n/a                                                              |
+--------------------+------------------------------------------------------------------+
| **Allowed values** | Any `sregex expression <regex.html#os-match-or-sregex-syntax>`_  |
+--------------------+------------------------------------------------------------------+

time
^^^^

Time that the event was generated.

+--------------------+----------------------------------------------------------------------+
| **Default Value**  | n/a                                                                  |
+--------------------+----------------------------------------------------------------------+
| **Allowed values** | Any time range (hh:mm-hh:mm, hh:mm am-hh:mm pm, hh-hh, hh am-hh pm)  |
+--------------------+----------------------------------------------------------------------+

weekday
^^^^^^^

Week day that the event was generated.

+--------------------+-------------------------------------+
| **Default Value**  | n/a                                 |
+--------------------+-------------------------------------+
| **Allowed values** | monday - sunday, weekdays, weekends |
+--------------------+-------------------------------------+

Here is a real example used in Wazuh on how to use this option:

  .. code-block:: xml

    <rule id="17102" level="9">
      <if_group>authentication_success</if_group>
      <weekday>weekends</weekday>
      <description>Successful login during weekend.</description>
    </rule>

id
^^^

Any ID (decoded as the ID).

+--------------------+------------------------------------------------------------------+
| **Default Value**  | n/a                                                              |
+--------------------+------------------------------------------------------------------+
| **Allowed values** | Any `sregex expression <regex.html#os-match-or-sregex-syntax>`_  |
+--------------------+------------------------------------------------------------------+

url
^^^

Any URL (decoded as the URL).

+--------------------+------------------------------------------------------------------+
| **Default Value**  | n/a                                                              |
+--------------------+------------------------------------------------------------------+
| **Allowed values** | Any `sregex expression <regex.html#os-match-or-sregex-syntax>`_  |
+--------------------+------------------------------------------------------------------+

location
^^^^^^^^

.. versionadded:: 3.5.0

The event extended location of the incoming event.

+--------------------+------------------------------------------------------------------+
| **Default Value**  | n/a                                                              |
+--------------------+------------------------------------------------------------------+
| **Allowed values** | Any `sregex expression <regex.html#os-match-or-sregex-syntax>`_  |
+--------------------+------------------------------------------------------------------+

The following components use a static location:

+----------------------+------------------------+
| **Component**        | **Location**           |
+----------------------+------------------------+
| Windows Eventchannel | EventChannel           |
+----------------------+------------------------+
| Windows Eventlog     | WinEvtLog              |
+----------------------+------------------------+
| FIM (Syscheck)       | syscheck               |
+----------------------+------------------------+
| Rootcheck            | rootcheck              |
+----------------------+------------------------+
| Syscollector         | syscollector           |
+----------------------+------------------------+
| Vuln Detector        | vulnerability-detector |
+----------------------+------------------------+
| Azure Logs           | azure-logs             |
+----------------------+------------------------+
| AWS S3 integration   | aws-s3                 |
+----------------------+------------------------+
| Docker integration   | Wazuh-Docker           |
+----------------------+------------------------+
| Osquery integration  | osquery                |
+----------------------+------------------------+
| OpenSCAP integration | open-scap              |
+----------------------+------------------------+
| CIS-CAT integration  | wodle_cis-cat          |
+----------------------+------------------------+

As a example, if we would want to see the incoming events from osquery:

  .. code-block:: xml

    <rule id="24000" level="3">
      <location>osquery$</location>
      <description>osquery message</description>
    </rule>

action
^^^^^^

Any action (decoded as the ACTION).

+--------------------+---------------------------------------------+
| **Default Value**  | n/a                                         |
+--------------------+---------------------------------------------+
| **Allowed values** | Any event action (deny, drop, accept, etc)  |
+--------------------+---------------------------------------------+

if_sid
^^^^^^

Matches if the ID has matched.

+--------------------+-------------+
| **Default Value**  | n/a         |
+--------------------+-------------+
| **Allowed values** | Any rule id |
+--------------------+-------------+

if_group
^^^^^^^^

Matches if the group has matched before.

+--------------------+-----------+
| **Default Value**  | n/a       |
+--------------------+-----------+
| **Allowed values** | Any Group |
+--------------------+-----------+

if_level
^^^^^^^^

Matches if the level has matched before.

+--------------------+------------------------+
| **Default Value**  | n/a                    |
+--------------------+------------------------+
| **Allowed values** | Any level from 1 to 16 |
+--------------------+------------------------+

if_matched_sid
^^^^^^^^^^^^^^

Matches if an alert of the defined ID has been triggered in a set number of seconds.

This option is used in conjunction with frequency and timeframe.

+--------------------+-------------+
| **Default Value**  | n/a         |
+--------------------+-------------+
| **Allowed values** | Any rule id |
+--------------------+-------------+

.. note::
  Rules at level 0 are discarded immediately and will not be used with the if_matched_rules. The level must be at least 1, but the <no_log> option can be added to the rule to make sure it does not get logged.

if_matched_group
^^^^^^^^^^^^^^^^

Matches if an alert of the defined group has been triggered in a set number of seconds.

This option is used in conjunction with frequency and timeframe.

+--------------------+-----------+
| **Default Value**  | n/a       |
+--------------------+-----------+
| **Allowed values** | Any Group |
+--------------------+-----------+

same_id
^^^^^^^

Specifies that the decoded id must be the same.
This option is used in conjunction with frequency and timeframe.

+--------------------+--------------------+
| **Example of use** | <same_id />        |
+--------------------+--------------------+

same_source_ip
^^^^^^^^^^^^^^

Specifies that the decoded source ip must be the same.
This option is used in conjunction with frequency and timeframe.

+--------------------+--------------------+
| **Example of use** | <same_source_ip /> |
+--------------------+--------------------+

same_src_port
^^^^^^^^^^^^^

Specifies that the decoded source port must be the same.
This option is used in conjunction with frequency and timeframe.

+--------------------+--------------------+
| **Example of use** | <same_src_port />  |
+--------------------+--------------------+

same_dst_port
^^^^^^^^^^^^^

Specifies that the decoded destination port must be the same.
This option is used in conjunction with frequency and timeframe.

+--------------------+--------------------+
| **Example of use** | <same_dst_port />  |
+--------------------+--------------------+

same_location
^^^^^^^^^^^^^

Specifies that the location must be the same.
This option is used in conjunction with frequency and timeframe.

+--------------------+--------------------+
| **Example of use** | <same_location />  |
+--------------------+--------------------+

same_user
^^^^^^^^^

Specifies that the decoded user must be the same.
This option is used in conjunction with frequency and timeframe.

+--------------------+--------------------+
| **Example of use** | <same_user />      |
+--------------------+--------------------+

same_field
^^^^^^^^^^

.. versionadded:: 3.9.0

``<same_field>`` and ``<not_same_field>`` are options that match when the value of a dynamic field from an incoming event is the same as the one of a previous event which matched the same rule, or backwards in the ``<not_same_field>`` case.

+--------------------+-----------+
| **Default Value**  | n/a       |
+--------------------+-----------+
| **Allowed values** | Any filed |
+--------------------+-----------+

Use case:

.. code-block:: xml

  <rule id="100001" level="3">
    <if_sid>221</if_sid>
    <field name="netinfo.iface.name">ens33</field>
    <description>Testing interface alert</description>
  </rule>

  <rule id="100002" level="7" frequency="3" timeframe="300">
    <if_matched_sid>100001</if_matched_sid>
    <same_field>netinfo.iface.name</same_field>
    <same_field>netinfo.iface.mac</same_field>
    <not_same_field>netinfo.iface.rx_bytes</not_same_field>
    <options>no_full_log</options>
    <description>Testing options for correlating repeated fields</description>
  </rule>

Rule 100002 matches when the third network inventory scan reports the same MAC address for the interface ``ens33`` but the amount of received packets has changed between events. 

different_url
^^^^^^^^^^^^^

Specifies that the decoded url must be different.
This option is used in conjunction with frequency and timeframe.

+--------------------+--------------------+
| **Example of use** | <different_url />  |
+--------------------+--------------------+

different_srcgeoip
^^^^^^^^^^^^^^^^^^

Specifies that the source geoip location must be different.
This option is used in conjunction with frequency and timeframe.

+--------------------+------------------------+
| **Example of use** | <different_srcgeoip /> |
+--------------------+------------------------+

Example:

  As a example of this last options, here is a rule that integrates a couple of them:

    .. code-block:: xml
      
      <rule id=100005 level="0">
        <match> Could not open /var/ossec </match>
        <same_user />
        <different_srcgeoip />
        <same_dst_port />
      </rule>

  That rule filters when the same ``user`` tries to open the folder ``/var/ossec`` from a different ``ip`` and using the same ``port``, but returns an error.

description
^^^^^^^^^^^

Used to add a description to a rule so it makes more clear and readable its funcionality.
This option apports more readable information for the users so is usually added to the rules.

+--------------------+------------+
| **Default Value**  | n/a        |
+--------------------+------------+
| **Allowed values** | Any string |
+--------------------+------------+

Examples:

  .. code-block:: xml

    <rule id="100009" level="1">
      <regex>(\d+.\d+.\d+.\d+)</regex>
      <description> Rule to match IPs </description>
    </rule>

  A rule description will be shown when decoded as:
  
  .. code-block:: bash

    **Phase 3: Completed filtering (rules)
    Rule id: '100009'
    Level: '1'
    Description: 'Rule to match IPs'
    ...
      
.. note:: Since Wazuh v3.3.0 it is possible to include any decoded field (static or dynamic) to the description message.

list
^^^^

Perform a CDB lookup using an ossec list.  This is a fast on disk database which will always find keys within two seeks of the file.

+--------------------+-------------------------------------------------------------------------------------------------------------------+
| **Default Value**  | n/a                                                                                                               |
+--------------------+-------------------------------------------------------------------------------------------------------------------+
| **Allowed values** | Path to the CDB file to be used for lookup from the OSSEC directory.Must also be included in the ossec.conf file. |
+--------------------+-------------------------------------------------------------------------------------------------------------------+

+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
| Attribute       | Description                                                                                                                       |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
| **field**       | key in the CDB: srcip, srcport, dstip, dstport, extra_data, user, url, id, hostname, program_name, status, action, dynamic field. |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
| **lookup**      | match_key               | key to search within the cdb and will match if they key is present. Default.                            |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
|                 | not_match_key           | key to search and will match if it is not present in the database.                                      |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
|                 | match_key_value         | searched for in the cdb. It will be compared with regex from attribute check_value.                     |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
|                 | address_match_key       | IP and the key to search within the cdb and will match if they key is present.                          |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
|                 | not_address_match_key   | IP the key to search and will match if it IS NOT present in the database                                |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
|                 | address_match_key_value | IP to search in the cdb. It will be compared with regex from attribute check_value.                     |
+-----------------+-------------------------+---------------------------------------------------------------------------------------------------------+
| **check_value** | regex for matching on the value pulled out of the cdb when using types: address_match_key_value, match_key_value                  |
+-----------------+-----------------------------------------------------------------------------------------------------------------------------------+

info
^^^^

Extra information may be added through the following attributes:

+--------------------+------------+
| **Default Value**  | n/a        |
+--------------------+------------+
| **Allowed values** | Any string |
+--------------------+------------+

+-----------+----------------+-----------------------------------------------------------------------------------------------------------+
| Attribute | Allowed values | Description                                                                                               |
+-----------+----------------+-----------------------------------------------------------------------------------------------------------+
| type      | **text**       | This is the default when no type is selected. Additional,information about the alert/event.               |
+           +----------------+-----------------------------------------------------------------------------------------------------------+
|           | **link**       | Link to more information about the alert/event.                                                           |
+           +----------------+-----------------------------------------------------------------------------------------------------------+
|           | **cve**        | The CVE Number related to this alert/event.                                                               |
+           +----------------+-----------------------------------------------------------------------------------------------------------+
|           | **ovsdb**      | The osvdb id related to this alert/event.                                                                 |
+-----------+----------------+-----------------------------------------------------------------------------------------------------------+

.. _rules_options:

options
^^^^^^^

Additional rule options.

+--------------------+-----------------------------------------------------+
| Attribute          | Description                                         |
+====================+=====================================================+
| **alert_by_email** | Always alert by email.                              |
+--------------------+-----------------------------------------------------+
| **no_email_alert** | Never alert by email.                               |
+--------------------+-----------------------------------------------------+
| **no_log**         | Do not log this alert.                              |
+--------------------+-----------------------------------------------------+
| **no_full_log**    | Do not include the ``full_log`` field in the alert. |
+--------------------+-----------------------------------------------------+
| **no_counter**     | Omit field ``rule.firedtimes`` in the JSON alert.   |
+--------------------+-----------------------------------------------------+

Example:

  .. code-block:: xml

    <rule id="9800" level="8">
      <match>illegal user|invalid user</match>
      <description>sshd: Attempt to login using a non-existent user</description>
      <options>no_log</options>
      <options>alert_by_email</options>
    </rule>

.. note::
  Use one ``<options>`` tag for each value added.

.. _rules_check_diff:

check_diff
^^^^^^^^^^

Used to determine when the output of a command changes.

+--------------------+--------------------+
| **Example of use** | <check_diff />     |
+--------------------+--------------------+

group
^^^^^

Add additional groups to the alert. Groups are optional tags added to alerts.

They can be used by other rules by using if_group or if_matched_group, or by alert parsing tools to categorize alerts.

Groups are variables that define a behaviour. When an alert includes that group label, this behaviour will occur.

Example:

  .. code-block:: xml

    <group name="osquery,">

      <rule id="24000" level="3">
        <location>osquery$</location>
        <description>osquery: $(osquery.pack) query result</description>
      </rule>

    </group>

Now, every rule included in this ``<group> </group>`` block will belong to that group.

So if we add the next line to that rule:

  .. code-block:: xml

    <group>osquery_monitoring,</group>

The rule will belong to the group ``osquery`` and ``osquery_monitoring`` at the same time.

Also, after have declared the ``osquery`` group, other rules can be added to this group by adding them the line:

  .. code-block:: xml

    <group>osquery,</group>

This is a very useful option to improve the filtering and ordering of rules.

+--------------------+------------+
| **Default Value**  | n/a        |
+--------------------+------------+
| **Allowed values** | Any String |
+--------------------+------------+

status
^^^^^^

Declares the actual status of a rule.

+--------------------+----------------------------------------------+
| **Default Value**  | n/a                                          |
+--------------------+----------------------------------------------+
| **Allowed values** | started, aborted, succedeed, failed, lost... |
+--------------------+----------------------------------------------+

var
^^^

Defines a variable that may be used in any place of the same file.

+----------------+------------------------+
| Attribute      | Value                  |
+================+========================+
| **name**       | Name for the variable. |
+----------------+------------------------+

Example:

  .. code-block:: xml

     <var name="joe_folder">/home/joe/</var>

      <group name="local,">

        <rule id="100001" level="5">
          <if_sid>550</if_sid>
          <field name="file">^$joe_folder</field>
          <description>A Joe's file was modified.</description>
          <group>ossec,pci_dss_10.6.1,gpg13_10.1,gdpr_IV_35.7.d,</group>
        </rule>

    </group>

BAD_WORDS
^^^^^^^^^

<var name="BAD_WORDS">error|warning|failure</var>

``BAD_WORDS`` is a very used case of ``<var>`` option.

Is used to include many words in the same variable. Later, this variable can be matched into the decoders to check if any of those words are in a caught event.

Example:

  .. code-block:: xml

    <var name="BAD_WORDS">error| warning |failure</var>

    <group name="syslog,errors,">
      <rule id="100300" level="2">
        <match>$BAD_WORDS</match>
        <description>Error found.</description>
      </rule>
    </group>
