.. _ossec_puppet:

OSSEC deployment with Puppet
============================

Puppet master installation
--------------------------

Before we get started with Puppet, check the following network requirements:

- Private network DNS: Forward and reverse DNS must be configured, and every server must have a unique hostname. If you do not have DNS configured, you must use your hosts file for name resolution. We will assume that you will use your private network for communication within your infrastructure.

+ Firewall open ports: The Puppet master must be reachable on port 8140.

Installation on CentOS
^^^^^^^^^^^^^^^^^^^^^^

Install your Yum repository, and puppet-server package, for your Enterprise Linux distribution. For example, for EL7: ::

   $ sudo rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
   $ sudo yum install puppetserver


Installation on Debian
^^^^^^^^^^^^^^^^^^^^^^

To install your Puppet master on Debian/Ubuntu systems, we first need to add our distribution repository. This can be done, downloading and installing a package named ``puppetlabs-release-distribution.deb`` where "distribution" needs to be substituted by your distribution codename (e.g. wheezy, jessie, trusty, utopic). See below the commands to install the Puppet master package for a "jessie" distribution: ::

   $ wget https://apt.puppetlabs.com/puppetlabs-release-pc1-trusty.deb
   $ sudo dpkg -i puppetlabs-release-pc1-trusty.deb
   $ sudo apt-get update && apt-get install puppetserver

Memory Allocation
^^^^^^^^^^^^^^^^^

By default, Puppet Server will be configured to use 2GB of RAM. However, if you want to experiment with Puppet Server on a VM, you can safely allocate as little as 512MB of memory. To change the Puppet Server memory allocation, you can edit the init config file.

  * ``/etc/sysconfig/puppetserver`` -- RHEL
  * ``/etc/default/puppetserver`` -- Debian

Replace 2g with the amount of memory you want to allocate to Puppet Server. For example, to allocate 1GB of memory, use ``JAVA_ARGS="-Xms1g -Xmx1g"``; for 512MB, use ``JAVA_ARGS="-Xms512m -Xmx512m"``.

Configuration
^^^^^^^^^^^^^

Configure ``/etc/puppetlabs/puppet/puppet.conf`` adding the ``dns_alt_names`` line to the ``[main]`` section, and replacing ``puppet.example.com`` with your own FQDN: ::

   [main]
   dns_alt_names = puppet,puppet.example.com

.. note:: If found in the configuration file, remove the line ``templatedir=$confdir/templates``, which has been deprecated.

Then, restart your Puppet master to apply changes: ::

   $ sudo service puppetserver start

PuppetDB installation
---------------------

After configuring your Puppet master to run on Apache with Passenger, the next step is to add Puppet DB so that you can take advantage of exported resources, as well as have a central storage place for Puppet facts and catalogs.

Installation on CentOS
^^^^^^^^^^^^^^^^^^^^^^
::

   $ sudo rpm -Uvh https://download.postgresql.org/pub/repos/yum/9.4/redhat/rhel-7-x86_64/pgdg-centos94-9.4-3.noarch.rpm
   $ yum install puppetdb-terminus.noarch puppetdb postgresql94-server postgresql94 postgresql94-contrib.x86_64
   $ sudo /usr/pgsql-9.4/bin/postgresql94-setup initdb
   $ systemctl start postgresql-9.4
   $ systemctl enable postgresql-9.4

Installation on Debian
^^^^^^^^^^^^^^^^^^^^^^
::

  $ sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" >> /etc/apt/sources.list.d/pgdg.list
  $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
    sudo apt-key add -
  $ sudo apt-get update
  $ apt-get install puppetdb-terminus puppetdb postgresql-9.4 postgresql-contrib-9.4

Configuration
^^^^^^^^^^^^^

The next step is to edit ``pg_hba.conf`` and modify the METHOD to ``md5`` in the next two lines

::

  /var/lib/pgsql/9.4/data/pg_hba.conf -- CentOS

::

  # IPv4 local connections:
  host    all             all             127.0.0.1/32            ``md5``
  # IPv6 local connections:
  host    all             all             ::1/128                 ``md5``

Create a PostgreSQL user and database: ::

   # su - postgres
   $ createuser -DRSP puppetdb
   $ createdb -O puppetdb puppetdb

The user is created so that it cannot create databases (-D), or roles (-R) and doesn’t have superuser privileges (-S). It’ll prompt for a password (-P). Let’s assume a password of "yourpassword"” has been used. The database is created and owned (-O) by the puppetdb user.

Test the database access and create the extension pg_trgm: ::

   # psql -h 127.0.0.1 -p 5432 -U puppetdb -W puppetdb
   Password for user puppetdb:
   psql (8.4.13)
   Type "help" for help.

   puppetdb=> CREATE EXTENSION pg_trgm;
   puppetdb=> \q

Configure ``/etc/puppetlabs/puppetdb/conf.d/database.ini``: ::

   [database]
   classname = org.postgresql.Driver
   subprotocol = postgresql
   subname = //127.0.0.1:5432/puppetdb
   username = puppetdb
   password = yourpassword
   log-slow-statements = 10

Create ``/etc/puppetlabs/puppet/puppetdb.conf``: ::

   [main]
   server_urls = https://puppetdb.example.com:8081

Create ``/etc/puppetlabs/puppet/routes.yaml``: ::

   ---
   master:
     facts:
       terminus: puppetdb
       cache: yaml

Finally, update ``/etc/puppetlabs/puppet/puppet.conf``: ::

   [master]
    storeconfigs = true
    storeconfigs_backend = puppetdb

Once all steps are completed, restart your Puppet master and run ``puppet agent --test``: ::

   $ puppet agent --test

Now PuppetDB is working.

Puppet agents installation
--------------------------

In this section we assume you have already installed APT and Yum Puppet repositories.

Installation on CentOS
^^^^^^^^^^^^^^^^^^^^^^
::

   $ sudo yum install puppet
   $ sudo puppet resource package puppet ensure=latest

Installation on Debian
^^^^^^^^^^^^^^^^^^^^^^
::

   $ sudo apt-get install puppet
   $ sudo apt-get update
   $ sudo puppet resource package puppet ensure=latest

Configuration
^^^^^^^^^^^^^

Add the server value to the ``[main]`` section of the node’s ``/etc/puppet/puppet.conf`` file, replacing ``puppet.example.com`` with your Puppet master’s FQDN::

   [main]
   server = puppet.example.com

Restart the Puppet service::

   $ service puppet restart

Puppet certificates
-------------------

Run Puppet agent to generate a certificate for the Puppet master to sign: ::

   $ sudo puppet agent -t

Log into to your Puppet master, and list the certificates that need approval: ::

   $ sudo puppet cert list

It should output a list with your node’s hostname.

Approve the certificate, replacing ``hostname.example.com`` with your agent node’s name: ::

   $ sudo puppet cert sign hostname.example.com

Back on the Puppet agent node, run the puppet agent again: ::

   $ sudo puppet agent -t

.. note:: Remember the Private Network DNS is a requisite for the correct certificate sign.

OSSEC Puppet module
-------------------

.. note:: This Puppet module has been authored by Nicolas Zin, and updated by Jonathan Gazeley and Michael Porter. Wazuh has forked it with the purpose of maintaining it. Thank you to the authors for the contribution.

Download and install OSSEC module from Puppet Forge: ::

   $ sudo puppet module install wazuh-ossec
   Notice: Preparing to install into /etc/puppet/modules ...
   Notice: Downloading from https://forgeapi.puppetlabs.com ...
   Notice: Installing -- do not interrupt ...
   /etc/puppet/modules
   └─┬ wazuh-ossec (v2.0.1)
     ├── jfryman-selinux (v0.2.5)
     ├── puppetlabs-apt (v2.2.0)
     ├── puppetlabs-concat (v1.2.4)
     ├── puppetlabs-stdlib (v4.9.0)
     └── stahnma-epel (v1.1.1)

This module installs and configures OSSEC HIDS agent and manager.

The manager is configured by installing the ``ossec::server`` class, and using optionally:

 - ``ossec::command``: to define active/response command (like ``firewall-drop.sh``).
 - ``ossec::activeresponse``: to link rules to active/response commands.
 - ``ossec::addlog``: to define additional log files to monitor.

Example
^^^^^^
Here is an example of a manifest ``ossec.pp``:

OSSEC manager: ::


  node "server.yourhost.com" {
     class { 'ossec::server':
       mailserver_ip => 'localhost',
       ossec_emailto => ['user@mycompany.com'],
       use_mysql => true,
       mysql_hostname => '127.0.0.1',
       mysql_name => 'ossec',
       mysql_password => 'yourpassword',
       mysql_username  => 'ossec',
     }

     ossec::command { 'firewallblock':
       command_name       => 'firewall-drop',
       command_executable => 'firewall-drop.sh',
       command_expect     => 'srcip'
     }

     ossec::activeresponse { 'blockWebattack':
        command_name => 'firewall-drop',
        ar_level     => 9,
        ar_rules_id  => [31153,31151],
        ar_repeated_offenders => '30,60,120'
     }

     ossec::addlog { 'monitorLogFile':
       logfile => '/var/log/secure',
       logtype => 'syslog'
     }

    class { '::mysql::server':
      root_password           => 'yourpassword',
      remove_default_accounts => true,
    }

    mysql::db { 'ossec':
      user     => 'ossec',
      password => 'yourpassword',
      host     => 'localhost',
      grant    => ['ALL'],
      sql      => '/var/ossec/contrib/sqlschema/mysql.schema'
    }
  }

OSSEC agent: ::

   node "client.yourhost.com" {

   class { "ossec::client":
     ossec_server_ip => "192.168.209.166"
   }

   }

Reference
^^^^^^^^^

OSSEC manager class
"""""""""""""""""""

class ossec::server
 - ``$mailserver_ip``: SMTP mail server.
 - ``$ossec_emailfrom`` (default: ``ossec@${domain}``: Email "from".
 - ``$ossec_emailto``: Email "to". ``['user1@mycompany.com','user2@mycompany.com']``
 - ``$ossec_active_response`` (default: ``true``): Enable/disable active-response (both on manager and agent).
 - ``ossec_server_port`` (default: '1514'): Port to allow communication between manager and agents.
 - ``$ossec_global_host_information_level`` (default: 8): Alerting level for the events generated by the host change monitor (from 0 to 16).
 - ``$ossec_global_stat_level``: (default: 8): Alerting level for the events generated by the statistical analysis (from 0 to 16).
 - ``$ossec_email_alert_level``: (default: 7): It correspond to a threshold (from 0 to 156 to sort alert send by email. Some alerts circumvent this threshold (when they have ``alert_email`` option).
 - ``$ossec_emailnotification``: (default: yes): Whether to send email notifications.
 - ``$ossec_prefilter`` : (default: ``false``) Command to run to prevent prelinking from creating false positives. ``This option can potentially impact performance negatively. The configured command will be run for each and every file checked.``
 - ``$local_decoder_template``:  (default: ``ossec/local_decoder.xml.erb``)
 - ``$local_rules_template``:   (default: ``ossec/local_rules.xml.erb``)
 - ``$manage_repo`` (default: ``true``): Install Ossec through Wazuh repositories.
 - ``$manage_epel_repo`` (default: ``true``): Install epel repo and inotify-tools
 - ``$manage_paths`` (default: ``[ {'path' => '/etc,/usr/bin,/usr/sbin', 'report_changes' => 'no', 'realtime' => 'no'}, {'path' => '/bin,/sbin', 'report_changes' => 'yes', 'realtime' => 'yes'} ]``): Follow the instructions bellow.
 - ``$ossec_white_list``: Allow white listing of IP addresses.
 - ``$manage_client_keys``: (default: ``true``): Manage client keys option.
 - ``$ossec_auto_ignore``: (default: ``yes``): Specifies if syscheck will ignore files that change too often (after the third change)
 - ``$use_mysql``: (default: ``false``). Set to ``true`` to enable database integration for alerts and other outputs.
 - ``mariadb``: (default: ``false``). Set to ``true`` to enable to use mariadb instead of mysql.
 - ``$mysql_hostname``: MySQL hostname.
 - ``$mysql_name``: MySQL Database name.
 - ``$mysql_password``: MySQL password.
 - ``$mysql_username``: MySQL username.
 - ``$syslog_output``: (default: ``false``).
 - ``$syslog_output_server``: (default: ``undef``).
 - ``$syslog_output_format``: (default: ``undef``).
 - ``$ossec_extra_rules_config``: To use it, after enabling the Wazuh ruleset (either manually or via the automated script), take a look at the changes made to the ossec.conf file. You will need to put these same changes into the "$ossec_extra_rules_config" array parameter when calling the ossec::server class.
 - ``$ossec_email_maxperhour``: (default: ``12``): Global Configuration with a larger maximum emails per hour
 - ``$ossec_email_idsname``: (default: ``undef``)
 - ``$server_package_version``: (default: ``installed``) Modified client.pp and server.pp to accept package versions as parameter.
 - ``$ossec_service_provider``: (default: ``$::ossec::params::ossec_service_provider``) Set service provider to Redhat on Redhat systems.
 - ``$ossec_rootcheck_frequency``: (default: ``36000``) Frequency that the rootcheck is going to be executed (in seconds).
 - ``$ossec_rootcheck_checkports``: (default: ``true``) Look for the presence of hidden ports.
 - ``$ossec_rootcheck_checkfiles``: (default: ``true``) Scan the whole filesystem looking for unusual files and permission problems.
 - ``$ossec_conf_template``: (default: ``ossec/10_ossec.conf.erb```) Allow to use a custom ossec.conf in the manager.

Consequently, if you add or remove any of the Wazuh rules later on, you'll need to ensure to add/remove the appropriate bits in the "$ossec_extra_rules_config" array parameter as well.

function ossec::email_alert
 - ``$alert_email``: Email to send to.
 - ``$alert_group``: (default: ``false``): Array of name of rules group.

.. note:: No email will be send below the global ``$ossec_email_alert_level``.

function ossec::command
 - ``$command_name``: Human readable name for ``ossec::activeresponse`` usage.
 - ``$command_executable``: Name of the executable. OSSEC comes preloaded with ``disable-account.sh``, ``host-deny.sh``, ``ipfw.sh``, ``pf.sh``, ``route-null.sh``, ``firewall-drop.sh``, ``ipfw_mac.sh``, ``ossec-tweeter.sh``, ``restart-ossec.sh``.
 - ``$command_expect`` (default: ``srcip``).
 - ``$timeout_allowed`` (default: ``true``).

function ossec::activeresponse
 - ``$command_name``.
 - ``$ar_location`` (default: ``local``): It can be set to ``local``,``server``,``defined-agent``,``all``.
 - ``$ar_level`` (default: 7): Can take values between 0 and 16.
 - ``$ar_rules_id`` (default: ``[]``): List of rules ID.
 - ``$ar_timeout`` (default: 300): Usually active reponse blocks for a certain amount of time.
 - ``$ar_repeated_offenders`` (default: empty): A comma separated list of increasing timeouts in minutes for repeat offenders. There can be a maximum of 5 entries.
function ossec::addlog
 - ``$log_name``.
 - ``$agent_log``: (default: ``false``)
 - ``$logfile`` /path/to/log/file.
 - ``$logtype`` (default: syslog): The OSSEC ``log_format`` of the file.

OSSEC agent class
"""""""""""""""""

 - ``$ossec_server_ip``: IP of the server.
 - ``$ossec_server_hostname``: Hostname of the server.
 - ``ossec_server_port`` (default: '1514'): Port to allow communication between manager and agents.
 - ``$ossec_active_response`` (default: ``true``): Allows active response on this host.
 - ``$ossec_emailnotification`` (default: ``yes``): Whether to send email notifications or not.
 - ``$ossec_prefilter`` : (default: ``false``) Command to run to prevent prelinking from creating false positives. ``This option can potentially impact performance negatively. The configured command will be run for each and every file checked.``
 - ``$selinux`` (default: ``false``): Whether to install a SELinux policy to allow rotation of OSSEC logs.
 - ``agent_name`` (default: ``$::hostname``)
 - ``agent_ip_address`` (default: ``$::ipaddress``)
 - ``$manage_repo`` (default: ``true``): Install Ossec through Wazuh repositories.
 - ``manage_epel_repo`` (default: ``true``): Install epel repo and inotify-tools
 - ``$ossec_scanpaths`` (default: ``[]``): Agents can be Linux or Windows for this reason don't have ``ossec_scanpaths`` by default.
 - ``$manage_client_keys``: (default: ``true``): Manage client keys option.
 - ``ar_repeated_offenders``: (default: empty) A comma separated list of increasing timeouts in minutes for repeat offenders. There can be a maximum of 5 entries.
 - ``/local_decoder_template``: (default: $::ossec::params::service_has_status) Allow configurable service_has_status, default to params.
 - ``agent_package_version``: (default: ``installed``) Modified client.pp and server.pp to accept package versions as parameter.
 - ``agent_package_name``: (default: ``$::ossec::params::agent_package``) Override package for client installation.
 - ``agent_service_name``: (default: ``$::ossec::params::agent_service``) Override service for client installation.
 - ``ossec_service_provider``: (default: ``$::ossec::params::ossec_service_provider``) Set service provider to Redhat on Redhat systems.
 - ``$ossec_conf_template``: (default: ``ossec/10_ossec_agent.conf.erb```) Allow to use a custom ossec.conf in the agent.

function ossec::addlog
 - ``$log_name``.
 - ``$agent_log`` (default: ``false``)
 - ``$logfile`` /path/to/log/file.
 - ``$logtype`` (default: syslog): The OSSEC ``log_format`` of the file.

ossec_scanpaths configuration
"""""""""""""""""""""""""""""

Leaving this unconfigured will result in OSSEC using the module defaults. By default, it will monitor /etc, /usr/bin, /usr/sbin, /bin and /sbin on Ossec Server, with real time monitoring disabled and report_changes enabled.

To overwrite the defaults or add in new paths to scan, you can use hiera to overwrite the defaults.

To tell OSSEC to enable real time monitoring of the default paths:

ossec::server::ossec_scanpaths:
  - path: /etc
    report_changes: 'no'
    realtime: 'no'
  - path: /usr/bin
    report_changes: 'no'
    realtime: 'no'
  - path: /usr/sbin
    report_changes: 'no'
    realtime: 'no'
  - path: /bin
    report_changes: 'yes'
    realtime: 'yes'
  - path: /sbin
    report_changes: 'yes'
    realtime: 'yes'

**Note: Configuring the ossec_scanpaths variable will overwrite the defaults. i.e. if you want to add a new directory to monitor, you must also add the above default paths to be monitored.**
