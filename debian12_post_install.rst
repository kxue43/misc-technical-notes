Debian 12 Post-installation Steps
==========================================

.. contents:: Contents:
   :local:

This page covers the necessary steps after installing Debian 12.

In the following steps, we assume an initial system update has been performed, either via Terminal or GNOME Software.

Install Flakpak
----------------

.. code-block:: bash

   $ sudo apt install flatpak gnome-software-plugin-flatpak
   $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

Install NVIDIA Proprietary Driver
--------------------------------------

By default, Debian 12 uses the graphics card integrated to the CPU. This is usually enough
for most GUI applications, but not for games and is a waste of resource if a standalone NVIDIA
graphics card is available.

To check which graphics card is used, go to :menuselection:`Settings -->About --> Graphics`.

To see if standalone graphics card is installed, run the following command.

.. code-block:: bash

   $ lspci -nn | egrep -i "3d|display|vga"

If the outputs include something like

.. code-block::

   01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107 [GeForce GTX 1050] [10de:1c81] (rev a1)

it means the GeForce 1050 Nvidia graphics card is installed.

The rest of this section assume that the machine is of ``x86_64`` architecture and that the graphics card is 
GeForce 600 series or newer.

The installation process is essentially building and using a custom Linux kernel module by ourselves.
If the machine uses `Secure Boot`_, we need to either disable it or sign the resulting kernel
module by ourselves. Otherwise the GUI desktop environment would not start properly. We can still login from terminal
by hitting :kbd:`Ctr` + :kbd:`Alt` + :kbd:`F2` at the black screen with a flashing cursor though.

To disable Secure Boot, at the "GNU GRUB" bootloader menu page, go to "UEFI Settings".
Find "Secure Boot" and toggle it to disabled. See `here <Secure Boot_>`_ for more details.

First add the following line to the :file:`/etc/apt/sources.list` file, which adds the "contrib", "non-free" and
"non-free-firmware" software repositories::

   deb http://deb.debian.org/debian/ sid main contrib non-free non-free-firmware

Then install the :program:`nvidia-driver` package with necessary firmware.

.. code-block:: bash

   $ sudo apt update
   $ sudo apt install nvidia-driver firmware-misc-nonfree

This will build the ``nvidia`` kernel module for the OS, via the :program:`nvidia-kernel-dkms` package.

.. note:: Compared to :ref:`Fedora Akmods <Fedora_Akmods>`, Debian uses DKMS instead of its own tooling for
     building kernel modules.

On Debian 12 with GNOME desktop and NVIDIA proprietary driver, it is possible that X11 is still used as the
Windowing System and we don't get the Wayland option at the login screen. Follow all the steps in
`NvidiaGraphicsDrivers#Wayland`_ to enable the Wayland option. These steps do the following things:

* Enabling kernel modesetting with the NVIDIA driver.
* Installing the hibernate/suspend/resume helper scripts and enabling relevant :program:`systemctl` services.
* Ensuring that the ``PreserveVideoMemoryAllocations`` NVIDIA module parameter is turned on.

.. note:: On Fedora we use the :program:`xorg-x11-drv-nvidia-power` RPM package to install the hibernate/suspend/resume
   helper scripts, while on Debian we download the scripts and install manually.

Finally, **restart the system** to load the new driver.

Afterwards, to check the status of the NVIDIA driver, run the following command.

.. code-block:: bash

   $ nvidia-smi

To check the version number of NVIDIA driver as a kernel module, run the following.

.. code-block:: bash

   $ modinfo -F version nvidia-current

Install Software
------------------

Dash-to-Dock
~~~~~~~~~~~~~

In GNOME Software, install the DEB version of "Tweaks" and "Extensions". Install "Dash-to-Dock" by:

.. code-block:: bash

   $ sudo apt install gnome-shell-extension-dashtodock

Open the "Extensions" app and configure Dash-to-Dock settings.

VS Code
~~~~~~~~~~~~

VS Code is available as a DEB package in a Microsoft repository. We need to install the repo and the signing key.
Then the package :program:`code` can be installed and auto-updated as other packages via :program:`apt`.

Install the repo and the signing key.

.. code-block:: bash

   $ sudo apt-get install wget gpg
   $ wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
   $ sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
   $ sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
   $ rm -f packages.microsoft.gpg

Install the :program:`code` package.

.. code-block:: bash

   $ sudo apt install apt-transport-https
   $ sudo apt update
   $ sudo apt install code # or code-insiders

Firefox
~~~~~~~~~~~

First remove the Firefox that comes with the distro as a DEB package, because its version is usually way behind
the current stable release of Firefox.

.. code-block:: bash

   $ sudo apt remove firefox-esr

Then install the Flathub version of Firefox in GNOME Software.

Thunderbird
~~~~~~~~~~~~~

Install the Flathub version of Thunderbird in GNOME Software.

Perform :ref:`configure_thunderbird` to make Thunderbird run in Wayland native.

Enable backports Repository
-------------------------------

Open the :file:`sources.list` file by:

.. code-block:: bash

   $ sudo apt edit-sources

Append the following line to the bottom of the file::

   deb http://deb.debian.org/debian bookworm-backports main contrib non-free

Update :program:`apt` cache by:

.. code-block:: bash

   $ sudo apt update

To find the backport version of a package:

.. code-block:: bash

   $ sudo apt show <package-name> -a

There are two ways of installing a backport:

.. code-block:: bash

   $ sudo apt install <package-name>/<release-name>-backports
   $ sudo apt install <package-name>/<release-name>-backports dependency/<release-name>-backports

``<release-name>`` is something like "bookwork".

The first installs the backport package while preferring dependencies from stable. The second prefers dependencies
from backports.

To list all installed backports:

.. code-block:: bash

   $ sudo dpkg-query -W | grep '~bpo'

Reference: https://wiki.debian.org/Backports#Using_backports

.. note:: Some package does not have backport versions, even if its Debian version is way behind its stable release,
     e.g. :program:`pipx`.

.. warning:: Do not install too many packages from backports.debian.org archives.
     It may cause package dependency complications. 

.. _Secure Boot: https://wiki.debian.org/SecureBoot
.. _NvidiaGraphicsDrivers#Wayland: https://wiki.debian.org/NvidiaGraphicsDrivers#Wayland
