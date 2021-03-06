.. _ssh_auth:

=========
SSH Auth
=========

This guide will show you how to enable and use the SSH authentication for the OpenNebula CLI. Using this authentication method, users login to OpenNebula with a token encrypted with their private ssh keys.

Requirements
============

You don't need to install any additional software.

Considerations & Limitations
============================

With the current release, this authentication method is only valid to interact with OpenNebula using the CLI.

Configuration
=============

OpenNebula Configuration
------------------------

The Auth MAD and ssh authentication is enabled by default. In case it does not work make sure that the authentication method is in the list of enabled methods.

.. code::

    AUTH_MAD = [
        executable = "one_auth_mad",
        authn = "ssh,x509,ldap,server_cipher,server_x509"
    ]

There is an external plain user/password authentication driver, and existing accounts will keep working as usual.

Usage
=====

Create New Users
----------------

This authentication method uses standard ssh rsa keypairs for authentication. Users can create these files **if they don't exist** using this command:

.. code::

    newuser@frontend $ ssh-keygen -t rsa

OpenNebula commands look for the files generated at the standard location (``$HOME/.ssh/id_rsa``) so it is a good idea not to change the default path. It is also a good idea to protect the private key with a password.

The users requesting a new account have to generate a public key and send it to the administrator. The way to extract it is the following:

.. code::

    newuser@frontend $ oneuser key
    Enter PEM pass phrase:
    MIIBCAKCAQEApUO+JISjSf02rFVtDr1yar/34EoUoVETx0n+RqWNav+5wi+gHiPp3e03AfEkXzjDYi8F
    voS4a4456f1OUQlQddfyPECn59OeX8Zu4DH3gp1VUuDeeE8WJWyAzdK5hg6F+RdyP1pT26mnyunZB8Xd
    bll8seoIAQiOS6tlVfA8FrtwLGmdEETfttS9ukyGxw5vdTplse/fcam+r9AXBR06zjc77x+DbRFbXcgI
    1XIdpVrjCFL0fdN53L0aU7kTE9VNEXRxK8sPv1Nfx+FQWpX/HtH8ICs5WREsZGmXPAO/IkrSpMVg5taS
    jie9JAQOMesjFIwgTWBUh6cNXuYsQ/5wIwIBIw==

The string written to the console must be sent to the administrator, so the can create the new user in a similar way as the default user/password authentication users.

The following command will create a new user with username 'newuser', assuming that the previous public key is saved in the text file /tmp/pub\_key:

.. code::

    oneadmin@frontend $ oneuser create newuser --ssh --read-file /tmp/pub_key

Instead of using the ``–read-file`` option, the public key could be specified as the second parameter.

If the administrator has access to the user's **private ssh key**, he can create new users with the following command:

.. code::

    oneadmin@frontend $ oneuser create newuser --ssh --key /home/newuser/.ssh/id_rsa

Update Existing Users to SSH
----------------------------

You can change the authentication method of an existing user to SSH with the following commands:

.. code::

    oneadmin@frontend $ oneuser chauth <id|name> ssh

    oneadmin@frontend $ oneuser passwd <id|name> --ssh --read-file /tmp/pub_key

As with the ``create`` command, you can specify the public key as the second parameter, or use the user's private key with the ``–key`` option.

User Login
----------

Users must execute the 'oneuser login' command to generate a login token. The token will be stored in the $ONE\_AUTH environment variable. The command requires the OpenNebula username, and the authentication method (``–ssh`` in this case).

.. code::

    newuser@frontend $ oneuser login newuser --ssh

The default ssh key is assumed to be in ``~/.ssh/id_rsa``, otherwise the path can be specified with the ``–key`` option.

The generated token has a default **expiration time** of 10 hour. You can change that with the ``–time`` option.
