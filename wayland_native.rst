Run Applications in Wayland Native
=======================================

.. contents:: Contents:
   :local:

Background
-------------

Some applications do not start in Wayland native but use Xwayland by default, e.g. VS Code, Chrome, and the Flathub
version of Thunderbird. Xwayland is an Xorg emulation layer of Wayland. It provides applications access
to Xorg libraries, allowing programs that only run on Xorg to run on Wayland.

VS Code is an Electron app --- the visual presentation is handled by Electron, which in turn depends on Chromium.
By default, when running Debian 12 or Fedora 39 with Wayland, both VS Code and Chrome browser use Xwayland instead of
Wayland native. This is probably caused by the underlying Chromium component.

Running VS Code with Xwayland in a Wayland session on a machine with NVIDIA drivers is not the most smooth experience,
but running it in Wayland native makes things much better.

This page describes how to detect if a GUI app is running Xwayland, and how to run some apps in Wayland native.

Detect Xwayland Applications Visually
--------------------------------------

Run the following command to start :program:`xeyes` from a terminal. A pair of eyes will appear in a small window.

.. code-block:: bash

   $ xeyes

When moving the mouse inside another app window, if the eyes are following the cursor, it means the app is running
with Xwayland. Otherwise it's running Wayland native.

To end :command:`xeyes`, simply hit :kbd:`Ctr` + :kbd:`C` in the terminal window.

On Debian 12, :program:`xeyes` comes with the :program:`x11-apps` package.

Reference: https://wiki.archlinux.org/title/wayland#Detect_Xwayland_applications_visually

VS Code
--------

The following command from a terminal starts VS Code in Wayland native.

.. code-block:: bash

   $ code --enable-features=UseOzonePlatform --ozone-platform=wayland

The command line flags are not parsed by :program:`code` but forwarded to Electron/Chromium,
as the stdout messages suggest.

As a side effect, this causes some glitches for the VS Code icon in the dock --- it's a different icon from the app
itself and cannot be pinned to dock.

In ArchLinux, there is a way to use the flags by putting them in a config file. On Debian/Fedora that approach doesn't
seem to exist, but we can put the command above as an alias for :program:`code` in :file:`.zshrc`.

.. _configure_chrome:

Chrome
-------

* Go to ``chrome://flags`` in Chrome address bar.
* Find "Preferred Ozone platform" in the menu. Set it to "Wayland".
* Restart Chrome.

This makes Chrome run more smoothly.

.. _configure_thunderbird:

Flathub Thunderbird
--------------------

Thunderbird is not built upon Electron so the method for VS Code and Chrome does not apply.

To configure the Flathub version of Thunderbird, run the following command.

.. code-block:: bash

   $ flatpak override --user --env=MOZ_ENABLE_WAYLAND=1 --socket=wayland org.mozilla.Thunderbird
