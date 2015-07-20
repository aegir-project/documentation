Uninstalling Aegir
==================

There is no formal method for uninstalling Aegir, but there is also no real mystery either since the system tries to keep itself together in one location, typically /var/aegir.

Below are the steps to completely remove all traces of Aegir from your server. Also included are instructions allowing one to leave Aegir intact, but destroy its data, attempting to reset to an almost post-install setup.

Ideally, you would delete all sites and all platforms from the system first. That would take care of deleting all databases and users. But this can take some time and is annoying, so we provide instructions on how to destroy everything.

##WARNING

Obviously if you have any sites and platforms currently managed by Aegir, you would want to move them out of the /var/aegir area before you delete it! Ensure you have set up your sites and platforms elsewhere, with Apache vhost files, etc. in their typical locations, or at least out of harm's way before attempting this.

*WARNING: Performing these steps will remove Aegir from your server and may result in loss of data. Use with caution.*

Run these commands as a privileged user (such as root). We assume your Aegir installation resides in /var/aegir.

###Backup

You probably want to backup everything before trashing it.

Backup all you need to keep from the /var/aegir/ directory.

Backup your database server, we'll be also destroying that.
Uninstall script

###Uninstall script

Aegir 3.x comes with a hostmaster-uninstall script that allows you to automate good chunks of the steps below.

###Destroying the data

Aegir manages three types of data:

* configuration files - yes, we consider those as data
* site and platform files
* site databases

###Trashing configuration files

This step is optional if you are going to remove everything anyways.

    rm /var/aegir/.drush/*alias.drushrc.php
    rm -r /var/aegir/config/*

###Trashing site and platform files

At this point, you could still probably recover by verifying the platforms, which would recreate config files. At this point we start really destroying data.

    rm -r /var/aegir/platforms/*
    rm -r /var/aegir/hostmaster-*

###Drop the databases and db users

There are no generic instructions for this, since every system differs. Essentially you can perform this within a MySQL shell

    mysql -u root -p

Use the DROP DATABASE $databasename; syntax to drop databases. and DROP USER $user; to delete users, example:

    DROP DATABASE aegirexamplecom;
    DROP USER aegirexamplecom@localhost;

This removes the Aegir user, assuming you have not created any site. If you did, you may want to cleanup those sites if you are going to delete them anyways. To see a list of GRANTs that Aegir has made for your database users, you can use a command like the following:

    USE mysql;
    SELECT User,Host FROM user WHERE Host='localhost' AND User <> 'root' AND User <> 'debian-sys-maint';
    SELECT Host,Db,User FROM db WHERE User <> '';

In Debian, the following will remove all non-system users (which may include non-aegir users!!!).

*WARNING: DO NOT RUN THIS BEFORE CHECKING WHAT WILL BE DELETE BY RUNNING THE SELECT ABOVE!! THIS WILL DELETE ALL USERS FROM YOUR MYSQL DATABASE!!!*

    DELETE FROM db WHERE User <> '';
    DELETE FROM user WHERE Host='localhost' AND User <> 'root' AND User <> 'debian-sys-maint';
    FLUSH PRIVILEGES;

Then you will need to DROP DATABASE for every database.

Consult the [GRANT](http://dev.mysql.com/doc/refman/5.1/en/grant.html) and [DROP](http://dev.mysql.com/doc/refman/5.1/en/drop-user.html) documentation from MySQL for more information.

###Remove the aegir user's crontab

    crontab -r -u aegir

##Removing everything else

At this point, we have removed all data that Aegir manages and we would be ready to reinstall the frontend. If we want to remove all traces, however, there's more work to do.

###Remove /var/aegir

    rm -rf /var/aegir

###Delete the aegir user

    userdel aegir

This will also remove the user from the www-data group.

###Remove the user from sudoers

    visudo

Remove the line that looks something like

    aegir ALL=NOPASSWD: /usr/sbin/apache2ctl

Save and exit the file.

###Delete the symlink/include from Apache

Depending on your installation and OS this may vary.

Generally:

    rm /etc/apache2/conf.d/aegir.conf

If your Include statements were contained in a global system httpd.conf file or similar, you will need to remove these lines manually.

Restart Apache when you have completed this.
