Upgrade Guide
=============

[TOC]

Upgrading Aegir is designed to be as easy as it possibly can be, and regularly improves over time as the developers attempt to streamline the process for users.

Nonetheless, the upgrade process can seem a little daunting to users. This is mainly because some expectation arises that because Aegir is built on Drupal, and is made for managing Drupal sites, it seems reasonable to expect the upgrade process to be as straightforward as, say, upgrading a regular Drupal site.

Aegir is a powerful system that goes beyond a normal Drupal application by being split into two parts: a frontend (the browser-based control panel) and the backend (bits on the system in /var/aegir, such as Drush and Drush extensions), along with a system user on your server that runs command-line scripts and restarts services, and cronjobs.

Typically what this means is that when it comes time to upgrade Aegir, you not only have to upgrade the frontend site, but also update the components that reside in the 'backend'.

But don't panic! We have instructions and even a script to run, to take care of almost all of it for you, eliminating as much as possible the chance for human error.


Upgrade options
---------------

There are three options you can choose to upgrade your Aegir install when a new release comes up:

* Upgrades using Debian packages
* Upgrades using the upgrade.sh.txt script
* Manual upgrades


New release of Drupal core
--------------------------

Just as you would use the Migrate task in Aegir to upgrade one of your sites to a new copy of Drupal core, you can follow the UPGRADE.txt to upgrade your actual Aegir frontend system to a new copy of Drupal core too. The upgrade process (including the script) will always fetch the latest available copy of Drupal core to place the frontend on.

Currently you cannot upgrade the Aegir site itself from the frontend with a Migrate task as you do for your managed sites. This may change in the future as we have an open issue over at [Allow upgrade of hostmaster sites from the frontend](https://www.drupal.org/node/711746).


Major upgrade from Aegir 6.x-2.x
--------------------------------

Most of the instructions on this page are targetted to minor updates, e.g. Aegir 3.1 to 3.2.
When upgrading from Aegir 2.x there are a few extra precautions to take. But apart from that the normal upgrade options apply.

To avoid the upgrade breaking run these drush commands:
```
drush @hostmaster en --yes ctools
drush @hostmaster pm-disable --yes hosting_platform_pathauto
drush @hostmaster pm-disable --yes install_profile_api
```

The upgrade will complain about missing modules like `modalframe` and `jquery_update`. This is to be expected, and can be ignored.

This upgrade path is [tested by Jenkins](http://ci.aegirproject.org/view/Upgrades/job/U_aegir_6.x-2.x_to_7.x-3.x_upgrade/).

Manual Upgrade
--------------

This section outlines the requirements for doing a manual upgrade of Aegir to a new release. Before going ahead with this, you probably want to read the upgrade path and version-specific notes.

### Conventions & Tips

All instructions and in general all commands must be run as aegir user, so all permissions are always set correctly.

To become aegir user you can issue this command. Note you must run this as root or prefix with sudo.

    su -s /bin/sh aegir

Note that /bin/sh is an example. You may wish to instead use the shell of your choice, i.e /bin/bash

A standard umask of 022 is assumed. This is the default on most systems.

### Upgrading Overview

#### Upgrading drush

As part of your Aegir upgrade, you may need to upgrade Drush to the latest stable version compatible with the target Aegir release. Aegir 3.x works with Drush 6, 7 and 8. Note that to install and manage Drupal 8 sites, Drush 8+ is required.

#### Upgrading the backend

After drush is updated, you must then proceed to upgrade the backend. The reason for upgrading the backend first before the frontend, is that the frontend upgrade process is instigated by the backend using Drush Make. Thus you need to be running the new backend first in order to successfully produce a new frontend.

In general, we try to keep the backend and the frontend compatible with each other during release cycles. That is: provision 3.2 and hosting 3.3 will always be able to talk to each other, but hosting 3.x may have trouble talking to the 2.x backend, so you need to upgrade the backend first when you do a major upgrade (for example, 2.4 -> 3.0).

Bottomline: first you upgrade the backend, then the frontend.

#### Upgrading the frontend

Once your backend is upgraded, you can upgrade your frontend. Think of this as the backend fetching a new copy of Drupal core and the Hostmaster frontend application onto your server, and then moving the Aegir site's settings.php and other bits and pieces over to the new codebase.

#### The hostmaster-migrate command

The command will make sure the target directory is a valid Aegir install. If the directory does not exist, provision will use drush_make to fetch and assemble the correct version of the front end for the specific release of the backend you are running.

`drush hostmaster-migrate` will also update the crontab entry for the aegir user.

If you have customized your Aegir installation and are maintaining your own makefile, you can use the *--makefile* flag so the platform is created with another makefile than the default. Be warned that this may create problems if the makefile doesn't include the right Aegir modules.

If there is a newer release of Aegir than what you have installed, the hostmaster command will fetch the latest stable Drupal release.  If you have the latest stable version of Aegir already installed, this won't happen because Drupal core changes could affect the stability of the distribution. So it cannot simply be run whenever a new security release of core Drupal is made available (unless your instance was installed via the *unstable* channel, which is not recommended).

To upgrade Drupal core (say for a new security release) while running Aegir stable, this method should work, but make sure to test on a development instance before running on your production environment:

1. Become the Aegir user.
    * `sudo -Hsu aegir`
1. Upgrade Drupal core.
    * `drush @hostmaster pm-updatecode drupal`
1. Verify the upgraded platform.
    * `drush @platform_hostmaster provision-verify`
1. Verify the Hostmaster site.
    * `drush @hostmaster provision-verify`
1. Run database updates
    * `drush @hostmaster updatedb`


Upgrades with Debian packages
-----------------------------

### Regular update process

Make sure you have the Aegir repository in your sources.list, as per the [installation docs](../install#2-add-project-repositories).

If you want to upgrade all packages in one shot, use:

    apt-get update
    apt-get install aegir3

Note that you can upgrade Aegir in steps. Upgrading your backend to the Debian package should be as simple as running:

    apt-get install aegir3-provision

Upgrading the frontend to the Debian package works identically:

    apt-get install aegir3-hostmaster

As during install, you can use the DEBUG variable to run Drush in debugging mode:

    env DPKG_DEBUG=yes apt-get install aegir3-hostmaster

**Note on contrib modules**: some Aegir-specific contrib modules may have been installed on your Aegir deployment. This may fail if you are doing a major upgrade. In this case, you will probably want to disable those contrib modules.  Look for directories in `~/.drush` or `/usr/share/drush/commands` or other components of the Drush commandfiles search path, and move them out of the way.  After the upgrade, download the new versions of the modules that are compatible with the new Aegir release.

### Temporarily upgrading to the unstable Debian package

Occasionally, you might be facing a bug that has been fixed in Aegir's git repos, but hasn't made it into a new release yet. The Aegir Project's CI system automatically builds and uploads new Debian packages for each commit to one of the core repos. As such, you *could* download and install the .deb directly. Note that this is *not* recommended.

1. Ensure that the latest build is passing the [test suite](http://ci.aegirproject.org/view/Upgrades/job/U_aegir_7.x-3.x-stable-to-unstable-deb-package/).
2. Visit http://debian.aegirproject.org/pool/main/a/aegir3-provision/
3. Find the latest unstable release packages for aegir3-provision and aegir3-hostmaster (they'll be the ones with a long suffix).
4. Download the Provision package to your Aegir server (e.g., `wget aegir3-provision_3.7+438.10e2f5b_all.deb`)
5. Download the Hostmaster package to your Aegir server (e.g., `wget aegir3-hostmaster_3.7+438.10e2f5b_all.deb`)
6. Install the Provision package to your Aegir server (e.g., `sudo dpkg -i aegir3-provision_3.7+438.10e2f5b_all.deb`)
7. Install the Hostmaster package to your Aegir server (e.g., `sudo dpkg -i aegir3-hostmaster_3.7+438.10e2f5b_all.deb`)

The idea here is that you're only running the unstable version of Aegir until the next full release, at which point the normal update procedure (e.g., `apt-get install aegir3`) will install the next stable release.

### Upgrading from non-Debian installs

The Debian package supports migrating from existing installs. You will need to move `/var/aegir/.drush/provision` out of the way before going ahead:

    tar cfz /var/aegir/backups/provision.tgz /var/aegir/.drush/provision
    rm -rf /var/aegir/.drush/provision

You'll also need to go through steps 1 through 3 of [automatic install on Debian](../install/#1-ensure-requirements-are-satisfied).

Then just install the package as if you were installing from scratch.

### Custom distributions

If you have your own makefile, you can go ahead with the above process, but change the makefile to the one you want:

    echo debconf aegir/makefile string
    /var/aegir/makefiles/aegir/aegir-koumbit.make | debconf-set-selections
    apt-get install aegir3

Usually no questions are asked when upgrading - this allows you to specify the makefile path for your custom distribution of Aegir, even if you're upgrading.  It's also how you can switch distributions.

An example aegir-koumbit.make file could look like:

    core = 7.x
    api = 2
    
    includes[aegir] = "/usr/share/drush/commands/provision/aegir.make"
    
    projects[] = module_filter

#### Keeping custom distributions up to date

This section describes how to upgrade the code for custom distributions in between debian package upgrades. Here, we assume that Aegir is already installed through the debian package.

Create a new Aegir makefile, or update the custom Aegir makefile. (You can use the makefile in http://git.drupal.org/project/provision as a starting point.) Add modules/features/themes/libraries/whatever to the makefile. Ex:

    projects[] = token
    projects[] = views

Follow the manual upgrade path:

    drush @hostmaster hostmaster-migrate URL /path/to/platform --makefile='/path/to/makefile'

Hope it's all green (no errors). If there are errors, fix the makefile.

A new platform should exist, the new site should be in there, verifying. If the task is not running, check that hosting-queued is running. If it's not, restart it using `/etc/init.d/hosting-queued force-reload`. (Since hostmaster changed location, hosting-queued may have crashed at this point.)

When the debian package is upgraded, it should create a new hostmaster platform automatically. (You can test this with `dpkg --configure`.)

### Rolling back the upgrade

If something went wrong with the upgrade, you can rollback by deleting the hostmaster site and redeploying on the older platform:

    drush @hostmaster provision-delete
    vi ~/.drush/hostmaster.alias.drushrc.php

Then edit the alias to make the following changes:

1. change the platform to point to the older platform alias
2. change the platform root path
3. change the site path

During the upgrade, a backup was done end you need to find which backup file it was in the backups directory. Use this backup to redeploy the older platform:

    drush @hostmaster provision-deploy ~/backups/hostmaster.date.tgz

#### Recovering a Failed Upgrade

When using the [regular update process](#regular-update-process) and you encounter an error it leaves the debian package in a broken state.

First you need to figure out why it failed. For example is there a permission problem? Once you determined the issue correct it and clean up the environment.  It may be necessary to remove the new hostmaster platform. It is up to you to determine what state Aegir is in.

Next you need to fix the broken package by running the fix-broken parameter:

    apt-get install -f

This tells the debian package system to start through the process again.


Upgrades with upgrade.sh script
-------------------------------

This page describes the upgrade script in the Provision repository that tries to automate much of the upgrade process.

It is imperative that you read the version-specific upgrade notes before attempting to run the upgrade.sh script, as the script will assume you have your system set up appropriately to handle the upgrade process.

You can download the upgrade.sh script for Aegir 3.x with the following command:

    wget -O upgrade.sh 'http://cgit.drupalcode.org/provision/plain/upgrade.sh.txt?h=7.x-3.121'

Make sure you download it to somewhere that the aegir user can access in order to execute it.

You may need to edit the script to set any variables that are different from the defaults, for example to upgrade to a different Aegir version.

To do the upgrade, just run the script:

    su -s /bin/sh aegir -c "sh upgrade.sh"
