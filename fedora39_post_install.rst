Fedora 39 Post-installation Steps
====================================

.. contents:: Contents:
   :local:

This page covers the necessary steps after installing a fresh Fedora 39 distribution.
In particular, it deals with the cannot-wake-from-suspension issue caused by NVIDIA drivers.

In the following steps, we assume an initial system update has been performed, either via Terminal or GNOME Software.

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

.. _Fedora_install_nvidia:

Install NVIDIA Proprietary Drivers
----------------------------------------

For a fresh installation, if we go to :menuselection:`Settings -->About --> System Details`, we can see that
the "Graphics" field shows "NV137", which is the open-source ``nouveau`` driver for NVIDIA cards. This driver doesn't
work well and we need to replace it with NVIDIA proprietary drivers.

The following commands assume that RPM Fusion has been enabled.

.. code-block:: bash

   $ sudo dnf update
   $ sudo dnf install kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig
   $ sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda

The above installs the NVIDIA proprietary driver.

The following solves the cannot-wake-from-suspension issue (`RPM Fusion HOWTO NVIDIA#Suspend`_).
Use them only if the problem exists.

.. code-block:: bash

   $ sudo dnf install xorg-x11-drv-nvidia-power
   $ sudo systemctl enable nvidia-{suspend,resume,hibernate}

Use the following command to check if the driver has finished building.
When installing for the first time, the command should output the version of the driver such as ``545.29.06`` if yes and
"modinfo: ERROR: Module nvidia not found" otherwise (`RPM Fusion HOWTO NVIDIA#Install`_). When upgrading, it should
output the upgraded version number instead of the existing one.
The build process usually `takes about five minutes`_.

.. code-block:: bash

   $ modinfo -F version nvidia

After reboot, :menuselection:`Settings -->About --> System Details` should show something like "NVIDIA GeForce GTX 1050",
which means proprietary driver is in use. Waking up from suspension also works, for both Wayland and X11.

NVIDIA driver is a custom built kernel module.
Custom built kernel modules only work with `a specific kernel version`_, so they have to
be built again every time the kernel is upgraded. Maybe this rebuilt is done properly by a distro via its
package manager after kernel upgrade, or maybe not. Since we have the fallback NVIDIA driver ``nouveau`` blacklisted
on both Debian (automatically) and Fedora (manually), it might happen that display is not working after a kernel
upgrade. In this case, we should try :kbd:`Ctr` + :kbd:`Alt` + :kbd:`F1`/\ :kbd:`F2` to enter a terminal session
with the otherwise functioning OS and simply rebuilding the kernel module via CLI.

.. _Fedora_Akmods:

There are tools that automatically rebuild kernel modules after kernel upgrade, such as `Dynamic Kernel Module Support`_.
RPM Fusion has a similar tool `Kmods2`_, which enables shipping *precompiled* kernel modules for the latest kernels
released by Fedora as RPM packages. There's also the `Akmods`_ tool that automate the whole process. Therefore, if
NVIDIA proprietary drivers are installed on Fedora via RPM Fusion (i.e. using the steps above),
it should continue to work after kernel upgrade
without users' manual intervention. This is a good reason NOT to install drivers by downloading binaries from vendor
website directly.

Install Software
--------------------

VS Code
~~~~~~~~~~~

We install an RPM repository and install :program:`code` from the repo.

Install a Microsoft RPM repo.

.. code-block:: bash

   $ sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
   $ sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'

Install the :program:`code` package.

.. code-block:: bash

   $ dnf check-update
   $ sudo dnf install code # or code-insiders

Dash-to-Dock
~~~~~~~~~~~~~~~

In GNOME Software, install the RPM version of "Tweaks" and the Flathub version of "Extensions".
Install "Dash-to-Dock" by:

.. code-block:: bash

   $ sudo dnf install gnome-shell-extension-dash-to-dock

Open "Extensions" app and configure Dash-to-Dock feature.

DejaVu Sans Mono Font
~~~~~~~~~~~~~~~~~~~~~~~

In GNOME Software, search "DejaVu Sans Mono" and install the regular and bold fonts. Then run:

.. code-block:: bash

   $ sudo fc-cache -v

Configure Dual Boot
----------------------

If Fedora 39 was installed after Debian 12, Fedora's bootloader lists Debian, but not the other way around.
To add Fedora to Debian's bootloader, do the following **on Debian**.

* Edit :file:`/etc/default/grub`. Uncomment the line with the following content::

     #GRUB_DISABLE_OS_PROBER=false

* Run the following command.

  .. code-block:: 

     $ sudo update-grub
   
  Output should include a line like "Found Fedora Linux 39 (Workstation Edition) on /dev/sda4".

* Change boot sequence in BIOS setting. Move Debian in front of Fedora. Or :command:`sudo dnf install efibootmgr` and
  use :program:`efibootmgr` to do the same thing (`reference`_).

Blacklist Kernel Modules
---------------------------------

When dual booting Fedora 39 from the Debian 12 GRUB, the Wireless adapter driver ``ath9k`` reports error and 
NVIDIA drivers somehow couldn't be detected. Fedora is still able to boot, but reverts to using ``nouveau``
as the graphics card driver. We can blacklist the ``ath9k`` and ``nouveau`` kernel modules to deal with this.

.. tip:: ``nouveau`` is automatically blacklisted after installing NVIDIA proprietary drivers on Debian.

To find the kernel module corresponding to the Wireless adapter:

.. code-block:: bash

   $ lspci -k

To blacklist the ``ath9k`` kernel module, create a file :file:`/etc/modprobe.d/ath9k-blacklist.conf` as root and put
the following contents in it:

.. code-block:: 

   # There is something wrong with this driver for Wireless adapter.
   # It causes problem when Fedora is booted from Debian's GRUB.
   blacklist ath9k

To blacklist ``nouveau``, create the file :file:`/etc/modprobe.d/nouveau-blacklist.conf` with the following content:

.. code-block:: 

   # Try blacklisting nouveau to see if it solves the NVIDIA issue.
   blacklist nouveau

Then run the following to regenerate `initramfs`_.

.. code-block:: bash

   $ sudo dracut -f

.. warning:: :program:`dracut` is NOT the default tool to create initramfs on Debian.
     Checkout `Debian blacklisting kernel modules <debian_blacklisting_>`_ for how to do that on Debian.

After reboot, use the following command to verify that the two modules have been blacklisted.

.. code-block:: bash

   $ modprobe --showconfig | grep blacklist

Comments on Kernel Taints
----------------------------------

After the operations above, when booting Fedora 39, we still get the following messages on the start-up screen::

   kernel: nvidia: loading out-of-tree module taints kernel.
   kernel: nvidia: module license 'NVIDIA' taints kernel.
   kernel: nvidia: module verification failed: signature and/or required key missing - tainting kernel
   kernel: nvidia: module license taints kernel.
   kernel: nvidia-nvlink: Nvlink Core is being initialized, major device number 235
   kernel: nvidia 0000:01:00.0: vgaarb: VGA decodes changed: olddecodes=io+mem,decodes=none:owns=io+mem
   kernel: nvidia_uvm: module uses symbols nvUvmInterfaceDisableAccessCntr from proprietary module nvidia, inheriting taint.
   kernel: nvidia-uvm: Loaded the UVM driver, major device number 511.
   kernel: nvidia-modeset: Loading NVIDIA Kernel Mode Setting Driver for UNIX platforms  545.29.06  Thu Nov 16 01:47:29 UTC 2023
   kernel: [drm] [nvidia-drm] [GPU ID 0x00000100] Loading driver
   kernel: [drm] Initialized nvidia-drm 0.0.0 20160202 for 0000:01:00.0 on minor 2
   kernel: nvidia 0000:01:00.0: vgaarb: deactivate vga console
   kernel: fbcon: nvidia-drmdrmfb (fb0) is primary device
   kernel: nvidia 0000:01:00.0: [drm] fb0: nvidia-drmdrmfb frame buffer device

.. tip:: These are the logs form :program:`systemd-journald.service`. They can be viewed by the command:

   .. code-block:: bash
   
      $ sudo journalctl -k | grep nvidia

   The ``-k`` option shows kernel messages only. We can :program:`grep` by any regex.

These are just warning messages. The lines starting with "nvidia-modeset" indicates that the NVIDIA driver is working
properly. Fedora boots more slowly than Debian because they use `different initrd schemas`_.

Messages that contain "taints kernel" means NVIDIA drivers' license and "closed-sourceness" makes it impossible to
properly troubleshoot some kernel problems, hence "taint". "Out-of-tree" means NVIDIA driver source code is not part of
Linux kernel source code. "License" is self-explanatory. "Required key missing" is due to not signing the NVIDIA driver,
a custom build kernel module, but if Secure Boot is disabled the driver still works. See `tainted kernel`_ for more
information.

.. note:: If we didn't blacklist ``nouveau``, the kernel taints will cause Fedora to fall back to using it.

Comments on Using Dual Monitor
---------------------------------

On the back of my PC there are four video output connectors: one HDMI from the CPU integrated graphics card; one HDMI,
one DVI and one DisplayPort from the NVIDIA GeForce GTX 1050 graphics card.

When using two monitors connected to the two HDMI ports, both Debian 12 and Fedora 39 show that the integrated and
the NVIDIA graphics cards are in use together. To make both monitors use the NVIDIA card only, the solution
is to plug both to the NVIDIA card --- one via HDMI and one via DVI is OK. Sometimes the solution is not in software
configurations but in hardware arrangements.

Interestingly, when only one monitor was plugged to the NVIDIA card's HDMI port, both distros were able to display
with either graphics card.

.. _reference: https://linuxconfig.org/how-to-manage-efi-boot-manager-entries-on-linux
.. _initramfs: https://en.wikipedia.org/wiki/Initial_ramdisk
.. _debian_blacklisting: https://wiki.debian.org/KernelModuleBlacklisting
.. _different initrd schemas: https://en.wikipedia.org/wiki/Initial_ramdisk#Mount_preparations
.. _tainted kernel: https://unix.stackexchange.com/questions/118116/what-is-a-tainted-linux-kernel
.. _takes about five minutes: https://discussion.fedoraproject.org/t/nvidia-gpu-kernel-module-problem-after-latest-updates/75590/10
.. _a specific kernel version: https://discussion.fedoraproject.org/t/nvidia-gpu-kernel-module-problem-after-latest-updates/75590/7
.. _RPM Fusion HOWTO NVIDIA#Install: https://rpmfusion.org/Howto/NVIDIA#Installing_the_drivers
.. _RPM Fusion HOWTO NVIDIA#Suspend: https://rpmfusion.org/Howto/NVIDIA#Suspend
.. _Dynamic Kernel Module Support: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support
.. _Kmods2: https://rpmfusion.org/Packaging/KernelModules/Kmods2
.. _Akmods: https://rpmfusion.org/Packaging/KernelModules/Akmods
