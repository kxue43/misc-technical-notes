Nvidia Drivers on Debian 12
=================================

.. contents:: Contents:
   :local:

By default, Debian 12 uses the graphics card integrated to the CPU. This is usually enough
for most GUI applications, but not for games and is a waste of resource if a standalone graphics card is installed.

To check which graphics card is used, go to :menuselection:`Settings -->About --> Graphics`.

To see if standalone graphics card is installed, run the following command.

.. code-block:: bash

   $ lspci -nn | egrep -i "3d|display|vga"

If the outputs include something like::

   01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107 [GeForce GTX 1050] [10de:1c81] (rev a1)

it means the GeForce 1050 Nvidia graphics card is installed.

The rest of this page explains how to install :program:`nvidia-driver` and let the OS use it. The machine is assumed
to be ``x86_64`` architecture. The graphics card is assumed to be GeForce 600 series or newer.

Install :program:`nvidia-driver`
---------------------------------------

The installation process is essentially building and using a custom Linux kernel module by ourselves.
If the machine uses `Secure Boot`_, we need to either disable it or sign the resulting kernel
module by ourselves. Otherwise the GUI desktop environment would not start properly. We can still login from terminal
by hitting :kbd:`Ctr` + :kbd:`Alt` + :kbd:`F2` at the black screen with a flashing cursor though.
See :ref:`Disable Secure Boot <disable_secure_boot>` for the process.

First add the following line to the :file:`/etc/apt/sources.list` file, which adds the "contrib", "non-free" and
"non-free-firmware" software repositories::

   deb http://deb.debian.org/debian/ sid main contrib non-free non-free-firmware

Then install the :program:`nvidia-driver` package with necessary firmware.

.. code-block:: bash

   $ sudo apt update
   $ sudo apt install nvidia-driver firmware-misc-nonfree

This will build the ``nvidia`` kernel module for the OS, via the :program:`nvidia-kernel-dkms` package.

Finally, restart the system to load the new driver.

.. note:: Compared to :ref:`Fedora Akmods <Fedora_Akmods>`, Debian uses DKMS instead of its own tooling for
     building kernel modules.

.. _wayland_or_x11:

Wayland or X11
-----------------

Debian 12, without NVIDIA drivers installed, uses Wayland as the Windowing System. NVIDIA supports Wayland, but with
caveats. It is claimed that "in Debian 12/bookworm, almost all issues should be resolved and most Wayland sessions
should 'just work' with the 525-series driver". However, GUI applications still has varying degrees of adaption 
to the Wayland and NVIDIA combination. For example, *VS Code doesn't work well in this setting*.

.. tip:: After the enabling steps below, we can choose the right Windowing System at each login.

On GNOME desktops, without additional steps, X11 is used as the Windowing System after NVIDIA drivers are installed,
and we don't get the Wayland option at the login screen. Follow *all* the steps in `here <NVIDIA Wayland_>`_ to enable
the Wayland option.

.. _disable_secure_boot:

Disable Secure Boot
---------------------

* Enter the "GNU Grub" page.
* Go the "UEFI Settings".
* Find "Secure Boot" and toggle it to disabled.

.. _Secure Boot: https://wiki.debian.org/SecureBoot
.. _NVIDIA Wayland: https://wiki.debian.org/NvidiaGraphicsDrivers#Wayland
