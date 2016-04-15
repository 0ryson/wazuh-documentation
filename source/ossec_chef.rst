.. _ossec_chef:

OSSEC deployment with Chef
==========================

Chef environment elements
---------------------------

.. image:: images/chef/workstation-server-node.png
    :align: center
    :width: 100%

Typically, Chef is comprised of three elements - your workstation, a Chef server, and nodes.

  - Your ```workstation``` is the computer from which you author your cookbooks and administer your network. It's typically the machine you use everyday. Although you'll be configuring a Red Hat Enterprise Linux server, your workstation can be any OS you choose - be it Linux, Mac OS, or Windows, in our case we will focus in Linux (CentOS & Ubuntu).
  - ``Chef server`` acts as a central repository for your cookbooks as well as for information about every node it manages. For example, the Chef server knows a node's fully qualified domain name (FQDN) and its platform.
  -  A ``node`` is any computer that is managed by a Chef server. Every node has the Chef client installed on it. The Chef client talks to the Chef server. A node can be any physical or virtual machine in your network.


Chef server installation
--------------------------

Before we get started with Chef, check the following network requirements:

  * An x86_64 compatible system architecture; Red Hat Enterprise Linux and CentOS may require updates prior to installation
  * A resolvable hostname that is specified using a FQDN or an IP address
  * A connection to Network Time Protocol (NTP) to prevent clock drift
  * A local mail transfer agent that allows the Chef server to send email notifications
  * Using cron and the /etc/cron.d directory for periodic maintenance tasks
  * Disabling the Apache Qpid daemon on CentOS and Red Hat systems
  * Optional. A local user account under which services will run, a local user account for PostgreSQL, and a group account under which services will run. See https://docs.chef.io/release/server_12-5/install_server_pre.html#uids-and-gids for more information.
  * Firewall open ports: The Chef server must be reachable on port 80 and 443.

Installation on CentOS
^^^^^^^^^^^^^^^^^^^^^^

Download Chef server package from http://downloads.chef.io/chef-server/, for your Enterprise Linux distribution. For example, for EL7: ::

   $ sudo wget https://packages.chef.io/stable/el/7/chef-server-core-12.5.0-1.el7.x86_64.rpm
   $ sudo rpm -Uvh chef-server-core-12.5.0-1.el7.x86_64.rpm

After a few minutes, the Chef server will be installed.

Installation on Ubuntu
^^^^^^^^^^^^^^^^^^^^^^

Download Chef server package from http://downloads.chef.io/chef-server/, for your Enterprise Linux distribution. For example, for Ubuntu 14.04: ::

   $ wget https://packages.chef.io/stable/ubuntu/14.04/chef-server-core_12.5.0-1_amd64.deb
   $ sudo dpkg -i chef-server-core_12.5.0-1_amd64.deb

After a few minutes, the Chef server will be installed.

Configuration
-------------

Adding features to Chef server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To configure Chef server is very easy you need to run the next command::

  $ sudo chef-server-ctl reconfigure

Because the Chef server is composed of many different services that work together to create a functioning system, this step may take a few minutes to complete.

If you like use Chef management console to manage data bags, attributes, run-lists, roles, environments, and cookbooks from a web user interface run this command: ::

  $ sudo chef-server-ctl install chef-manage
  $ sudo chef-server-ctl reconfigure
  $ sudo chef-manage-ctl reconfigure

If you like use Reporting to keep track of what happens during every chef-client runs across all of the infrastructure being managed by Chef. Run Reporting with Chef management console to view reports from a web user interface: ::

  $ sudo chef-server-ctl install opscode-reporting
  $ sudo chef-server-ctl reconfigure
  $ sudo opscode-reporting-ctl reconfigure

Workstation configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^



