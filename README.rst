.. _TLP:
   http://linrunner.de/en/tlp/tlp.html

.. _TLP git repo:
   https://github.com/linrunner/TLP

.. _tlp-gentoo-additions:
   https://github.com/dywisor/tlp-gentoo-additions

.. _tlp-portage:
   git://github.com/dywisor/tlp-portage.git

.. _layman:
   http://wiki.gentoo.org/wiki/Layman

.. _tpacpi-bat:
   https://github.com/teleshoes/tpacpi-bat

.. _upstream documentation:
   http://www.linrunner.de/en/tlp/docs/tlp-configuration.html


=============
 tlp-portage
=============

Overlay for installing `TLP`_ on Gentoo/Funtoo/... systems.


Setup Instructions
==================

The following commands (``$ <command...>``) should be run as root.
It is assumed that your package manager is ``sys-apps/portage``.


#. Install `layman`_

   #. Enable the git USE flag for layman::

      $ mkdir /etc/portage/package.use
      $ echo "app-portage/layman git" >> /etc/portage/package.use/layman

   #. Install layman::

      $ emerge -a --noreplace ">=app-portage/layman-2"

#. Make sure that ``/etc/portage/make.conf`` has the follwing line::

      source /var/lib/layman/make.conf

   If you've just installed layman, simply run::

      $ echo "source /var/lib/layman/make.conf" >> /etc/portage/make.conf

#. Add the *tlp-portage* overlay with layman::

      $ layman --overlays="https://raw.github.com/dywisor/tlp-portage/maint/layman.xml" --fetch --add=tlp

#. **stable arch** only (amd64, x86): unmask *TLP*::

      $ mkdir /etc/portage/package.accept_keywords
      $ echo "app-laptop/tlp" > /etc/portage/package.accept_keywords/tlp

#. *(optional)* install/build kernel modules

   This is required for battery charge threshold control.

   * Thinkpads up to Core 2 (and Sandy Bridge partially)::

      $ emerge -a app-laptop/tp_smapi

   * more recent Thinkpads::

      $ emerge -a app-laptop/acpi_call

#. *(optional)* choose USE flags, for example::

      $ echo "app-laptop/tlp tlp_suggests" > /etc/portage/package.use/tlp

   See `USE flags`_ below for a full listing.

#. Install *TLP*::

      $ emerge -a app-laptop/tlp

#. Edit *TLP's* configuration file **/etc/conf.d/tlp**

   In contrast to other distributions, the config file is not in */etc/default*
   and **you have to enable TLP explicitly** by setting ``TLP_ENABLE=1``.

   See the `upstream documentation`_ for details.


#. Enable the *TLP* service

   **OpenRC** (most Gentoo users)::

      $ rc-update add tlp default

   **systemd**::

      $ systemctl enable tlp.service
      $ systemctl enable tlp-sleep.service

#. Enable or restart/reload the *acpid* service

#. Start **TLP**::

      $ tlp start

#. You might want to run ``tlp-stat`` to see if everything is OK so far



-----------
 USE flags
-----------

.. table:: USE flags accepted by app-laptop/tlp

   +--------------+--------------+---------+--------------------------------------+
   | flag         | recommended  | default | description                          |
   +==============+==============+=========+======================================+
   | tlp_suggests | yes          | no      | install all optional dependencies    |
   +--------------+--------------+---------+--------------------------------------+
   | rdw          | \-           | no      | install *TLP's* radio device wizard  |
   +--------------+--------------+---------+--------------------------------------+
   | pm-utils     | yes (OpenRC) | yes     | depend on ``sys-power/pm-utils``     |
   |              |              |         | (can only be deselected when         |
   |              | \- (systemd) |         | using systemd)                       |
   +--------------+--------------+---------+--------------------------------------+
   | tpacpi-\     | **yes**      | yes     | use the bundled version of           |
   | bundled      |              |         | `tpacpi-bat`_                        |
   |              |              |         |                                      |
   |              |              |         | Deselecting this flag                |
   |              |              |         | **disqualifies you from getting \    |
   |              |              |         | support upstream**                   |
   +--------------+--------------+---------+--------------------------------------+
   | laptop-\     | **no**       | no      | Allow parallel installation of       |
   | mode-\       |              |         | ``app-laptop/tlp`` and               |
   | tools        |              |         | ``app-laptop/laptop-mode-tools``.    |
   |              |              |         | Having both active at the same time  |
   |              |              |         | is not supported at all.             |
   +--------------+--------------+---------+--------------------------------------+


--------------------
 Random notes / FAQ
--------------------


Kernel config considerations
----------------------------

The following kernel options should be set to *y*:

* CONFIG_DMIID
* CONFIG_ACPI_PROC_EVENT
* CONFIG_POWER_SUPPLY
* CONFIG_ACPI_AC