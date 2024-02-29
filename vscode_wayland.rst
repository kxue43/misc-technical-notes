Running VS Code in Wayland Native
=====================================

.. contents:: Contents:
   :local:

VS Code is an Electron app --- the visual presentation is handled by Electron, which in turn depends on Chromium.

Xwayland is an Xorg emulation layer of Wayland. It provides applications access to Xorg libraries, allowing programs
that only run on Xorg to run on Wayland.

By default, when running Debian 12 with Wayland, both VS Code and Chrome browser use Xwayland instead of Wayland native.
This is probably caused by the underlying Chromium components.

As mentioned in :ref:`wayland_or_x11`, running VS Code with Xwayland in a Wayland session on a machine with NVIDIA
drivers is not the most smooth experience, but running VS Code in Wayland native makes things much better.

This page describes how to detect if a GUI app is running Xwayland, and how to run VS Code and Chrome in Wayland native.

Detect Xwayland Applications Visually
--------------------------------------

When using Debian under Wayland, to detect if a GUI app is running Xwayland, run

.. code-block:: bash

   $ xeyes

from a terminal. Then a pair of eyes appear in a small window. When moving the mouse inside another app window,
if the eyes are following the cursor, it means the app is running under Xwayland. Otherwise it's running Wayland native.

To end :command:`xeyes`, simply hit :kbd:`Ctr` + :kbd:`C` in the terminal window.

On Debian 12, :program:`xeyes` comes with the :program:`x11-apps` package.

Starting VS Code in Wayland Native
----------------------------------------

Debian 12, start VS Code from terminal by the following command.

.. code-block:: bash

   $ code --enable-features=UseOzonePlatform --ozone-platform=wayland

Then VS Code runs in Wayland native. The command line flags are not parsed by :program:`code` but forwarded to
Electron/Chromium, as the stdout messages suggest.

As a side effect, this causes some glitches for the VS Code icon in the deck --- it's a different icon from the app
itself and cannot be pinned to deck.

In ArchLinux, there is a way to use the flags by putting them in a config file. On Debian that approach doesn't seem
to exist, so we put the command above as an alias for :program:`code` in :file:`.zshrc`.

.. _configure_chrome:

Configuring Chrome to Run in Wayland Native
-----------------------------------------------

* Go to ``chrome://flags`` in Chrome address bar.
* Find "Preferred Ozone platform" in the menu. Set it to "Wayland".
* Restart.

This makes Chrome run more smoothly.

References:

* https://wiki.archlinux.org/title/wayland#Detect_Xwayland_applications_visually

Installing VS Code on Debian
------------------------------------

VS Code is available as a ``.deb`` package in a Microsoft repository. We need to install the repo and the signing key.
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

Installing VS Code on Fedora
--------------------------------

Similarly, on Fedora we can install an RPM repository and install :program:`code` from the repo.

Install a Microsoft RPM repo.

.. code-block:: bash

   $ sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
   $ sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'

Install the :program:`code` package.

.. code-block:: bash

   $ dnf check-update
   $ sudo dnf install code # or code-insiders
