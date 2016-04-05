.. _manual_authd:

ossec-auth with password
===========================

.. versionadded:: v1.1

The ``ossec-authd`` will automatically add an agent to the manager and provide a
new key to the agent. ``agent-auth`` is the client application used by agents to 
connect with ossec-authd.

Wazuh HIDS provides **password-based authentication** for agents. The server
looks for a defined password at file ``/var/ossec/etc/authd.pass``. If a
password isn't found, a random one is generated and shown on the console.

**Duplicated IPs** are not longer allowed. So if there's an attempt to add two
agents with the same IP, ossec-authd will fail and report it through an alert.

.. _`ossec-authd`: http://ossec-docs.readthedocs.org/en/latest/programs/ossec-authd.html
.. _`client-auth`: http://ossec-docs.readthedocs.org/en/latest/programs/agent-auth.html

Configuration
-------------

On server-side
^^^^^^^^^^^^^^

``ossec-authd -[Vhdtin] [-f sec] [-g group] [-D dir] [-p port] [-v path] [-x path] [-k path]``

.. seealso::
    For a complete description of every option, please read `OSSEC documentation: ossec-authd`_.

    .. _`OSSEC documentation: ossec-authd`: http://ossec-docs.readthedocs.org/en/latest/programs/ossec-authd.html


New options:

-i              Register agent with client's IP instead of *any*.
-f <seconds>    Remove old agents with the same IP if they were not connected
                since <seconds>. It has only sense along with option ``-i``.
-P              Enable shared password authentication.

Option ``-f`` forces the insertion on IP collision, this means that if OSSEC 
finds another agent with the same IP, but it has not connected since the 
specified time, that agent will be deleted automatically and the new agent will 
be added. To force insertion always (regardless of the time of the last agent 
connection), use ``-f 0``.

On client-side
^^^^^^^^^^^^^^

``agent-auth -[Vhdt] [-g group] [-D dir] [-m IP address] [-p port] [-A name] [-v path] [-x path] [-k path] [-P pass]``

.. seealso::
    For a complete description of every option, please read `OSSEC documentation: agent-auth`_.

    .. _`OSSEC documentation: agent-auth`: http://ossec-docs.readthedocs.org/en/latest/programs/agent-auth.html


New options:

-P <password>    Use the specified password instead of search for it at
                 ``authd.pass``.

If a password is not provided at file neither on console, the client will
connect with the server without password (insecure mode).

Data backup
-----------

Before OSSEC removes an agent by forcing, it will backup the data about the old
agent at ``/var/ossec/backup/agents/<id> <name> <ip> <delete timestamp>``, in a
new folder with the agent's name and IP, and the current timestamp. The saved data is the following:

- Agent's operating system.
- Version of the agent.
- Timestamp when it was added.
- Syscheck database.
- Rootcheck database.

.. seealso::
    There is a compile option that allows a new agent to **inherit the ID** of 
    the agent that was removed by forcing insertion. To learn more about this, 
    please read :ref:`manual_reuse_id`.
    