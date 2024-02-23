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

The command above can be made into an alias in :file:`.zshrc`.

Configuring Chrome to Run in Wayland Native
-----------------------------------------------

* Go to ``chrome://flags`` in Chrome address bar.
* Find "Preferred Ozone platform" in the menu. Set it to "Wayland".
* Restart.

This makes Chrome run more smoothly.

References:

* https://wiki.archlinux.org/title/wayland#Detect_Xwayland_applications_visually
