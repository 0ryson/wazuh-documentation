.. _docker_installation:

Docker installation
===============================

Docker requires a 64-bit installation regardless of your CentOS or Debian version. Also, your kernel must be 3.10 at minimum.

To check your current kernel version, open a terminal and use ``uname -r`` to display your kernel version::

   $ uname -r
   3.10.0-229.el7.x86_64

Run the Docker installation script. ::

   $ curl -sSL https://get.docker.com/ | sh

If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like: ::

  $ sudo usermod -aG docker your-user

.. note:: Remember that you will have to log out and log in for this to take effect!
