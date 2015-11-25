Provision ACL
=============

[TOC]

This section documents the [Provision ACL extension](https://www.drupal.org/project/provisionacl) to Aegir which allows more granular access control over your sites files and directories. While this extension does not ship with Aegir by default, it can be extremely useful in securing access to resources on the CLI.

Install instructions
--------------------

Most (if not all) of these commands will have to be run as root (or using sudo, etc.)

### Install provisionacl

First, download and install provisionacl:

    drush dl provisionacl
    drush cc drush

### Install ACL support package

ACL support usually requires the installation of a system package:

    apt-get install acl

### Enable ACLs on your filesystem

To enable ACLs on you filesystem, it will have to be re-mounted with ACL support:

    mount -o remount,acl /

Here we assume everything is under the root (`/`) filesystem here, otherwise run this command for every filesystem Aegir will work on (e.g. `/srv`, `/var` or `/home`).

You also need to edit your `/etc/fstab` for this configuration to survive reboots.

### Create a UNIX group

In this case we choose a group called "devs" but you can choose another name.

    groupadd devs

### Add users to the group

Add one or more UNIX users that you want to give access to that group. For an existing user, this would look like:

    usermod -a -G devs <username>

For a new user, this would look like:

    useradd -G devs <username>

### Create a client

Create a client (should be called "devs" for this example) in the frontend at `/node/add/client`. The client name (in the front-end) and the user group (on the system) should match. Each new client that you want to grant such CLI access to will require its own user group.

### Create a site

Create a site for the client in the frontend at `/node/add/site`.


What ProvisionACL does
----------------------

When the site is installed, members of the "devs" group will be able to write to the sites' directories (e.g. upload files and modules) and run drush commands on the site (yes, including site aliases, although see caveats below).

This works also for existing sites; make sure you create a group matching the internal name of the existing client and reverify the site.


LDAP integration
----------------

Provisionacl supports LDAP groups as well. Ensure that an LDAP client is running and that the 'aegir' user can see the LDAP-provided groups:

    getent groups

You may need to restart the Name Service Cache Daemon (nscd):

    /etc/init.d/nscd restart


ProvisionACL API
----------------

ACL support can be integrated into contrib or custom Aegir extensions.

To change ACLs on files, you should use something like this:

    if (function_exists('provisionacl_set_acl')) {
      provisionacl_files_acls(d()->site_path . '/mysettings.php');
    }

You can optionnally pass a group as an argument, but it will guess that from the client name of the site. Also note that this will raise a drush error if setfacl fails, but just set a warning if the group doesn't exist.


Caveats (ie. what it does not do)
---------------------------------

Giving shell access to users in Aegir is still insecure, see [this upstream issue](https://drupal.org/node/762138).

We may refactor this into Aegir core in the future, but in the meantime this should provide a good workaround for the limitations of the existing permission system.

You may need to change your $HOME variable for aliases to work, because of [this bug in Drush](https://drupal.org/node/1104438). Example:

    env HOME=/var/aegir drush @hostmaster cc all

See also [this post](http://community.aegirproject.org/node/494/index.html) for context and design.
