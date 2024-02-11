Set Up SSH Server to Allow Remote-in
========================================

This page describes how to set up a Ubuntu machine as an SSH server
to allow others to remote in.

SSH Server
------------

First install :program:`openssh-server`.

.. code-block:: bash

   $ sudo apt-get install openssh-server

Afterwards check if the ``sshd`` service is active.

.. code-block:: bash

   $ sudo systemctl status sshd

SSH Client
-------------

First generate an SSH key pair.

.. code-block:: bash

   $ ssh-keygen -t ed25519 -C <label>

Add the *public key* to the SSH server, this time using password authentication
of the user.

.. code-block:: bash

   $ ssh-copy-id -i $PUBLIC_KEY_PATH ${USER}@${HOST}

Put the following section in :file:`~/.ssh/config`. Replace ``<***>`` with the values
of the same-named environment variables above::

   Host <HOST>
     User <USER>
     IdentityFile <PUBLIC_KEY_PATH>
     ForwardAgent yes

Remote into the SSH server with the following command.

.. code-block:: bash

   $ ssh ${USER}@${HOST}

Optionally Disable SSH Password Authentication
-----------------------------------------------

After setting up SSH Key authentication for one user, we can optionally disable
SSH password authentication on the server altogether.

Open the file :file:`/etc/ssh/sshd_config`. Find the line with::

   #PasswordAuthentication yes

Change it to::

   PasswordAuthentication no

Then restart the ``sshd`` service.

.. code-block:: bash

   $ sudo systemctl restart sshd

References
--------------

* `OpenSSH configuration <OpenSSH_configuring_>`_
* `SSH Key authentication`_

.. _OpenSSH_configuring: https://help.ubuntu.com/community/SSH/OpenSSH/Configuring
.. _SSH Key authentication: https://code.visualstudio.com/docs/remote/troubleshooting#_quick-start-using-ssh-keys
