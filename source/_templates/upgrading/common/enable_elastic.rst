.. Copyright (C) 2020 Wazuh, Inc.

.. tabs::


  .. group-tab:: Systemd


    .. code-block:: console

      # systemctl daemon-reload
      # systemctl start elasticsearch.service



  .. group-tab:: SysV Init

    Choose one option according to the OS used:

      .. tabs::

        .. group-tab:: Debian based OS

          .. code-block:: console

            # update-rc.d elasticsearch defaults 95 10
            # service elasticsearch start

        .. group-tab:: RPM based OS

          .. code-block:: console

            # chkconfig --add elasticsearch
            # service elasticsearch start

.. End of include file
