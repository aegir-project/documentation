Importing sites into Aegir
==========================

[TOC]

It is possible to import existing sites into the Aegir hosting system, so that they can be managed by Aegir.

These instructions assume you've set up a new server with Aegir on it, and you want to import sites into Aegir from another server, or even from the same server, using backups.

There are three standard ways of importing sites into Aegir:

* using [Remote Import](#remote-import) - simplest and fastest, scriptable
* from any [single site](#importing-a-single-site-manually) manually - most reliable, can import even non-aegir sites, hard to automate
* from a [complete drupal install](#importing-a-complete-drupal-platform) - fairly reliable, can import non-aegir sites, could be automated but requires access to the database servers currently in use by the site

Remote Import
-------------

This Drupal module provides a UI for fetching sites from remote Aegir servers.

###Installation

Install this module like any other, and enable in the usual way.

###Usage

You'll need to add the remote Aegir server as any other server in the frontend,
selecting ONLY the 'hostmaster' remote service as you do so.

This means, of course, that you'll need to add your ssh key to this server and
set it up like other remote servers. Note that you don't need to install
anything other than the SSH key on this server.

A general guide to setting up SSH on remote servers can be found [here](http://aegir.readthedocs.org/en/3.x/usage/advanced/remote-servers/#ssh-keys).

Once you've set up your server in the frontend you should get a new menu item
called: 'Import remote sites' when viewing the server node. Click on that link
and follow the instructions there.


Importing a single site from another Aegir
------------------------------------------

If Remote Import isn't working for you (perhaps a broken site on the source server is blocking a site listing), you can pretty easily perform similar operations manually.

1. Install a site with the same name on the destination server.
2. Take a backup of the new site.
3. Disable the site on the source server (this generates a backup).
4. Use `scp` or `rsync` to copy the latest backup from the source server to destination.
5. Move the backup from the source over top of the one taken on the destination.
6. Restore the backup via the Aegir UI  (on the destination server).
7. Clear caches, rebuild registries and verify the new site.
8. (Optional) Point the DNS entry for the site to the new server.
9. (Optional) Login to the site, and perhaps fix some internally stored paths (i.e., CiviCRM's "Resource URL")


Importing a single site manually
--------------------------------

If you already have a platform setup on Aegir with EXACTLY the same codebase as your existing site, then you don't need to transfer the entire old codebase - you can just transfer the sites/example.com directory. However, you also need to make sure any dependencies on contrib modules that your site has, are covered on the codebase or Platform that you're importing it into.

In general it may be considered safer/less prone to confusing errors to [transfer the entire old codebase](#complete-drupal-install) into Aegir as a whole Platform, whereby the site will be imported automatically under that Platform. You can then migrate it in Aegir to one of your existing Platforms later.

Also, it is much faster to just "deploy" an Aegir backup than go through the manual procedure here, so you should generally follow that procedure instead of the one defined here unless you have a very hairy setup.

### 1. Create site resources

In order to import the site, you need to create a database and database user for the site, along with a directory in the platform.

The simplest way to do this is to create an empty site in the right platform and overwrite it. Through the commandline:

    $ drush provision-save @example.com --context_type=site --platform=@platform_d6 --uri=example.com --db_server=@server_localhost --client_name=admin

    $ drush provision-install @example.com

Through the UI, this can be done simply by creating a site in Create content.

If this is not working for some reason, you can create the mysql user and database manually using the following:

    mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, LOCK TABLES, CREATE TEMPORARY TABLES ON databasename.* TO 'databaseuser'@'localhost' IDENTIFIED BY 'password';

    mysql> CREATE DATABASE databasename;

### 2. Transfer files

You need to copy the sites/example.com directory from your old server to the new Aegir server.

To do this, first create a tarball of the site on the old server. Within the sites/example.com directory type:

    $ tar -zcvf example.com.tar.gz .

Then, on the Aegir server, you can create a directory for it in the codebase (platform) you want to put it under:

    $ cd /var/aegir/platforms/drupal-X.Y/sites
    $ mkdir example.com
    $ cd example.com
    $ wget http://example.com/example.com.tar.gz
    $ tar -zxvf example.com.tar.gz ./files ./modules ./libraries ./themes

Instead of using wget you could of course use curl, rsync, SCP or FTP to transfer the tarball onto the Aegir server.

In the above we assume that you have created an empty site as directed above, if not, you will also need to extract the settings.php file, so that last command should instead be simply:

    $ tar -zxvf example.com.tar.gz

Then you will need to modify the settings.php file to follow the user and database created in the first step. Again, this is not necessary if you let Aegir create the site for you.

Once the file is unpacked, check the ownership and group of each file by using ls -al. It should be either 'aegir:aegir' or 'aegir:www-data'. Change it if necessary. In particular, files that may have been uploaded on your site via modules like imagecache, upload, Profile pics etc, may need to have their ownership changed:

    $ sudo chown -R aegir:www-data $platform/sites/*/files

### 3. Transfer database

Make a backup of the database on your old server Backup and Migrate module is good for this, or you can use phpmyadmin or your favourite MySQL management tool. It's a good idea to truncate the cache, search and watchdog tables first to reduce the size of the database. The "Backup and Migrate" module does this for you.

If you are importing a site from a previous Aegir installation somewhere, such as from a backup tarball, the database.sql will already be within the tarball you unpacked. You can use this to import into a new database on your new server.

You can import the database with drush using the following command:

    $ drush @example.com sqlc < database.sql

### 4. Verify in Aegir

In the Aegir web interface click on the name of the platform you have added your site to. You can access the list of platforms via the 'Platforms' tab in the primary links menu.

Execute a new Verify task on this platform via the 'Verify' button in the task list.

Aegir will now re-verify the platform. In the process of this, it will discover your new site and spawn a new 'Import' task for the site.

Once it's done this (it may take a couple of cron runs to dispatch these tasks in the queue) the tasks should turn green.

At this point, presuming you have updated DNS or are overriding DNS via an entry in your /etc/hosts file to access the site via the new Aegir server, you can visit the site in a browser and check around to see that it has worked. Pay particular attention to any links in node content that pointed to paths referencing /sites/community.aegirproject.org if your site was not part of a multisite setup. (Drupal has a habit of storing these paths in the database, or they may have been hard-coded into nodes by users).

Aegir will not go through all your database and update all URLs, so some images or links may be broken. This will need to be done manually. Aegir does attempt to update paths stored in tables such as 'files' though.

It's a good idea to clear the caches, and you may need to get imagecache to rebuild its thumbnails if you use it.


Importing a single site using a remote drush alias and Git
--------------------------------

Drush can sync files and databases from a remote server when you have ssh access there.
We create a new platform and dummy site and overwrite it with the real data.

### Requirements

1. The Git integration module (Install via Hosting => Advanced)
2. The Extra tasks module (Install via Hosting => Advanced)
3. (optional) The Git pull task module (Install via Hosting => Advanced)
4. SSH public key access to the origin server.

### Prepare platform

*You can skip this is you already have a suitable platform on your server, just make sure it has all the modules that the imported site needs.*

1. Create a Git repository on your favaurite Git hosting site.
2. Clone the new repository on a workstation.
3. Copy the original document root into this new repository.
4. Add the lines below to the .gitignore file, and preserve the lines that Drupal core ships with.

```
# Aegir related code and config
sites/*/drushrc.php
sites/*/local.settings.php
sites/all/drush/drushrc.php
sites/sites.php
```

5. Commit and push
```
git add -A
git commit -m 'Initial import'
git push origin master
```

6. On your hostmaster site go to /node/add/platform and add the platform from the git repo.

7. If you have enabled the Git pull task module then you can set this new platform to use the 'Pull on URL Callback' method.
You the add the 'Pull Trigger URL' shown on the platform's edit page to you Git hosting site as Trigger/webhook.

### Setting up a Drush alias for the origin site.

Add a custom Drush alias for the origin site. Add the following to `~/.drush/aliases.drushrc.php`.
```
<?php
$aliases['originwww'] = array (
  'remote-host' => 'oldserver.example.com',
  'remote-user' => 'webuser',
  'root' => '/home/webuser/sites/example.com/www',
  'uri' => 'http://example.com/',
);
```



### Transfer the data.

These steps could potentially be done with the 'Sync task' module, but the commands below give a better insight into what is being done.

Database:
```
drush sql-sync @originwww @www.example.com
```

When the site was not a multisite before (it was in in sites/default) you can use this to set things strait in the database.
```
drush @www.example.com provision-update_uri www.example.com www.example.com
```

Files:
```
drush rsync @originwww:%files/ @www.example.com:%files
```




### Testing

Before changing the DNS you can test if the site behaves correctly on the new host by updateing the /etc/hosts file on your local machine to add the new IP for the site uri.

- login en check /admin/reports/status
- It's a good idea to clear the caches, and you may need to get imagecache to rebuild its thumbnails if you use it.



### Live

- (optional) repeat the earlier sync steps to get the latest data from the origin server, you might want to put that in maintenance mode first to be sure.
- Update the DNS
- Optionally enable SSL




Importing a complete Drupal platform
------------------------------------

This method of importing sites is often considered the safest whereby you transfer across an entire codebase containing Drupal core, and define it as a Platform first. This makes sure you bring any dependencies, contrib modules etc, in exactly the same version and configuration that is referenced in the database for the site.

### 1. Transfer the codebase

Create a tarball of the codebase on the old server. Within the drupal root directory type: tar -zcvf example.com.tar.gz * .htaccess

Note that we have to specify including the .htaccess because it isn't picked up by the * wildcard due to being prefixed with a full stop. It is important to bring along the .htaccess for a platform so that Aegir can consider these settings when it generates Apache configuration for this platform.

Then, on the new server you can create a directory for it in the codebase you want to put it under:

    $ cd /var/aegir/platforms
    $ mkdir your-new-platform
    $ cd your-new-platform
    $ wget http://example.com/example.com.tar.gz
    $ tar -zxvf example.com.tar.gz

Instead of using wget you could of course use SCP or FTP to transfer the tarball onto the Aegir server.

Once the file is unpacked, check the ownership and group of each file by using ls -al. It should be either 'aegir:aegir' or 'aegir:www-data'. Change it if necessary. We have to assume you have basic knowledge of Linux and POSIX permissions - we can't be responsible for teaching you sysadmin skills :)

In particular, files that may have been uploaded on your site via modules like
imagecache, upload, Profile pics etc, may need to have their ownership changed:
sudo chown -R aegir $platform/sites/\*/files

If your site previously resided in sites/default you need to move it because Aegir ignores the default directory (it has no way of understanding what URL this would be accessed by, so it is impossible to 'manage' it). Each site needs its own directory with the correct domain in typical Drupal multisite design:

    $ mv sites/defau1t sites/example.com

If you do this, you may need to also update your filepath's stored in the files table by running the following query on your database:

    mysql> UPDATE files SET filepath = REPLACE(filepath, 'sites/defau1t', 'sites/example.com');

There are many other places in the database that can suffer the same problem, the node_revisions table for example. If you have phpMyAdmin available you can easily search the entire database for 'sites/defau1t'. Select the database, then click the Search tab. Once you know which tables are affected you can run a variation of the above query to correct these records.

### 2. Transfer the Database

Note: This step is not necessary if you are moving your site from one directory on your server (e.g., /var/www/html) to a newly created Aegir directory on the same server (e.g., /var/aegir).

Make a backup of the database on your old server Backup and Migrate module is good for this, or you can use phpmyadmin or your favourite MySQL management tool. It's a good idea to truncate the cache, search and watchdog tables first to reduce the size of the database (Backup and Migrate does this for you).

Aegir does not provide support for importing databases with prefixed tables so it is best to remove prefixes form the database. However, a [workaround is available](https://www.drupal.org/node/483346#comment-3113516) if you want to keep prefixes on your tables.

Unlike traditional Drupal projects the database settings cannot be found in settings.php. The database credentials are stored in the Apache vhost config of the associated site. This is a security measure implemented by the Aegir project.

In the vhost directory on your old server, /var/aegir/config/server_master/apache/vhost.d/, there is a vhost file for every site on your server. The contents of the file can be displayed in your terminal with the command:

    $ cat thenameofthefile

At the top of the file you will find the credentials needed to create a database on the new server.

On your new server, manually create a new database and upload the .sql file from your backup.

Then create the mysql user that your site accesses the database as, and grant it all permissions on that database except 'GRANT'.

Connect to the database on the old server:

    $ mysql -u root -p

Create the new database:

    mysql> create database yourdatabasename

Add the user:

    mysql> CREATE USER 'yourdatabaseuser'@'localhost' identified by 'youruserpassword';

Give the necessary privileges:

    mysql> GRANT ALL PRIVILEGES ON yourdatabasename.* TO 'yourdatabaseuser'@'localhost';

Import the database:

    $ mysql -u yourdatabaseuser -p -h localhost yourdatabasename < thenameofthedatabase.sql

### 3. Setup Platform in Aegir

In the Aegir web interface click on 'Create Content' and 'Platform'. This step is often easiest when using the admin-menu on the top of your screen.

Enter the name you want to use for the platform (eg 'My Platform import'), and the path to the platform on the server (eg '/var/aegir/platforms/your-new-platform')

Click Save to submit the form, and Aegir will spawn a Verify task for this platform into the queue.

Once the task is complete this the task should turn green (click through to the homepage to refesh and check. If not, see 'Troubleshooting' below).

### 4. Import and Verify Sites In Aegir

Upon completion of the above Platform verification task, Aegir will discover your new site and spawn a new 'Import' task for the site.

Once it's done this (it may take a couple of cron runs to dispatch these tasks in the queue) the tasks should turn green (if not, see 'Troubleshooting' below).

At this point, presuming you have updated DNS or are overriding DNS via an entry in your /etc/hosts file to access the site via the new Aegir server, you can visit site in a browser and check around to see that it has worked. Pay particular attention to any links in node content that pointed to paths referencing /sites/community.aegirproject.org if your site was not part of a multisite setup. (Drupal has a habit of storing these paths in the database, or they may have been hard-coded into nodes by users).

It's a good idea to clear the caches, and you may need to get imagecache to rebuild its thumbnails if you use it.

### 5. Migrate To Another Platform

Now that you have your sites under Aegir's control you can take advantage of its power, and easily migrate them to another platform. In the event that your sites were on old versions of Drupal core and a new one is available and present as another Platform on your Aegir server, you can use the Migrate process to upgrade those sites onto the new core.

For details on how to Migrate sites, consult the [Migrate documentation](/usage/sites/tasks/#migratingupgrading-sites).
