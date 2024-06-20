Set Up SSH Server and Client
========================================

This page describes how to set up a Ubuntu machine as SSH server to allow clients to remote in.

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

Use SSH Agent Forwarding to Allow :command:`git clone` on Server
-------------------------------------------------------------------

When cloning a private GitHub repository over SSH *on the server*, the server needs access to the SSH private key.
It is not secure to place the SSH private key file directly on the server. The better option is to enable SSH agent
forwarding on the client and forward the private key from client to server when they are connected.

For example, to set up agent forwarding for `github.com`, put the following contents in :file:`~/.ssh/config`::

   Host <server_ip_v4>
   	ForwardAgent yes
   Host github.com
   	User git
   	AddKeysToAgent yes
   	IdentityFile <path_to_private_key_file>

The ``ForwardAgent yes`` option shares the client's private keys with the server whenever there is a connection,
so should be used with caution. In particular, it is best to use specific IP addresses or host names in the 
first rule, instead of using wildcard.

References
--------------

* `OpenSSH configuration <OpenSSH_configuring_>`_
* `SSH Key authentication`_
* `Troubleshooting SSH agent forwarding`_

.. _OpenSSH_configuring: https://help.ubuntu.com/community/SSH/OpenSSH/Configuring
.. _SSH Key authentication: https://code.visualstudio.com/docs/remote/troubleshooting#_quick-start-using-ssh-keys
.. _Troubleshooting SSH agent forwarding: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/using-ssh-agent-forwarding#troubleshooting-ssh-agent-forwarding
