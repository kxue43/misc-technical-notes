Fedora 39 Post-installation Steps
====================================

.. contents:: Contents:
   :local:

This page covers the necessary steps after installing a fresh Fedora 39 distribution.
In particular, it deals with the cannot-wake-from-suspension issue caused by NVIDIA drivers.

In the following steps, we assume an initial system update has been performed, either via Terminal or Gnome Software.

DNF Configuration
---------------------

Edit the file :file:`/etc/dnf/dnf.conf` and append the following contents to it.

.. code-block:: 

   # Added for speed
   fastestmirror=True
   max_parallel_downloads=10
   keepcache=True

Enable RPM Fusion
------------------

Run the following commands.

.. code-block:: bash

   $ sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
   $ sudo dnf groupupdate core

Add Flathub
------------

.. code-block:: bash

   $ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

Change Hostname
-----------------

.. code-block:: bash

   $ sudo hostnamectl set-hostname fedora-desktop

Install NVIDIA Proprietary Drivers
----------------------------------------

For a fresh installation, if we go to :menuselection:`Settings -->About --> System Details`, we can see that
the "Graphics" field shows "NV137", which is the opensource Nouveau driver for NVIDIA cards. This driver doesn't
work well and we need to replace it with NVIDIA proprietary drivers.

The following commands assume that RPM Fusion has been enabled.

.. code-block:: bash

   $ sudo dnf update
   $ sudo dnf install kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig
   $ sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda

The above installs the NVIDIA proprietary driver.

The following solves the cannot-wake-from-suspension issue (`NVIDIA official doc`_).

.. code-block:: bash

   $ sudo dnf install xorg-x11-drv-nvidia-power
   $ sudo systemctl enable nvidia-{suspend,resume,hibernate}

Now reboot the system. :menuselection:`Settings -->About --> System Details` should show something like
"NVIDIA GeForce GTX 1050", which means proprietary driver is in use. Waking up from suspension also works,
for both Wayland and X11.

Install Software
--------------------

Create a folder :file:`~/build-rpms` to save the RPM packages from which direct installation was performed.

For VS Code and Google Chrome, download the RPM packages from the official websites and perform
:command:`sudo dnf install` on them.

In Gnome Software, install the RPM version of "Tweaks" and the Flathub version of "Extensions".
Install "Dash-to-Dock" by:

.. code-block:: bash

   $ sudo dnf install gnome-shell-extension-dash-to-dock

Open "Extensions" app and configure Dash-to-Dock feature.

In Gnome Software, search "DejaVu Sans Mono" and install the regular and bold fonts. Then run:

.. code-block:: bash

   $ sudo fc-cache -v

Configure Chrome
------------------

Do the same this as in :ref:`configure_chrome`.

Configure Dual Boot
----------------------

If Fedora 39 was installed after Debian 12, Fedora's bootloader lists Debian, but not the other way around.
To add Fedora to Debian's bootloader, do the following on Debian.

* Edit :file:`/etc/default/grub`. Uncomment the line with the following content::

     #GRUB_DISABLE_OS_PROBER=false

* Run the following command.

  .. code-block:: 

     $ sudo update-grub
   
  Output should include a line like "Found Fedora Linux 39 (Workstation Edition) on /dev/sda4".

* Change boot sequence in BIOS setting. Move Debian in front of Fedora.

.. _NVIDIA official doc: https://rpmfusion.org/Howto/NVIDIA#Suspend
