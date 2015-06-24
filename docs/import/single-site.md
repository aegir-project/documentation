Importing a single site manually
================================

If you already have a platform setup on Aegir with EXACTLY the same codebase as your existing site, then you don't need to transfer the entire old codebase - you can just transfer the sites/example.com directory. However, you also need to make sure any dependencies on contrib modules that your site has, are covered on the codebase or Platform that you're importing it into.

In general it may be considered safer/less prone to confusing errors to [transfer the entire old codebase](complete-drupal-install) into Aegir as a whole Platform, whereby the site will be imported automatically under that Platform. You can then migrate it in Aegir to one of your existing Platforms later.

Also, it is much faster to just "deploy" an Aegir backup than go through the manual procedure here, so you should generally follow that procedure instead of the one defined here unless you have a very hairy setup.

To transer a site manually into Aegir, those are the steps you should follow:

Table of Contents

    1. Create the site database, user and directory
    2. Transfer the files
    3. Transfer the Database
    4. Verify In Aegir

1. Create the site database, user and directory

In order to import the site, you need to create a database and database user for the site, along with a directory in the platform.

The simplest way to do this is to create an empty site in the right platform and overwrite it. Through the commandline:

	drush provision-save @example.com --context_type=site --platform=@platform_d6 --uri=example.com --db_server=@server_localhost --client_name=admin

	drush provision-install @example.com

Through the UI, this can be done simply by creating a site in Create content.

If this is not working for some reason, you can create the mysql user and database manually using the following:

	GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, LOCK TABLES, CREATE TEMPORARY TABLES ON databasename.* TO 'databaseuser'@'localhost' IDENTIFIED BY 'password';

	CREATE DATABASE databasename;

2. Transfer the files

You need to copy the sites/example.com directory from your old server to the new Aegir server.

To do this, first create a tarball of the site on the old server. Within the sites/example.com directory type:

	tar -zcvf example.com.tar.gz .

Then, on the Aegir server, you can create a directory for it in the codebase (platform) you want to put it under:

	cd /var/aegir/platforms/drupal-X.Y/sites
	mkdir example.com
	cd example.com
	wget http://example.com/example.com.tar.gz
	tar -zxvf example.com.tar.gz ./files ./modules ./libraries ./themes

Instead of using wget you could of course use SCP or FTP to transfer the tarball onto the Aegir server.

In the above we assume that you have created an empty site as directed above, if not, you will also need to extract the settings.php file, so that last command should instead be simply:

	tar -zxvf example.com.tar.gz

Then you will need to modify the settings.php file to follow the user and database created in the first step. Again, this is not necessary if you let Aegir create the site for you.

Once the file is unpacked, check the ownership and group of each file by using ls -al. It should be either 'aegir:aegir' or 'aegir:www-data'. Change it if necessary. In particular, files that may have been uploaded on your site via modules like imagecache, upload, Profile pics etc, may need to have their ownership changed:

	sudo chown -R aegir:www-data $platform/sites/*/files

3. Transfer the Database

Make a backup of the database on your old server Backup and Migrate module is good for this, or you can use phpmyadmin or your favourite MySQL management tool. It's a good idea to truncate the cache, search and watchdog tables first to reduce the size of the database. The "Backup and Migrate" module does this for you.

If you are importing a site from a previous Aegir installation somewhere, such as from a backup tarball, the database.sql will already be within the tarball you unpacked. You can use this to import into a new database on your new server.

You can import the database with drush using the following command:

	drush @example.com sqlc < database.sql

4. Verify In Aegir

In the Aegir web interface click on the name of the platform you have added your site to. You can access the list of platforms via the 'Platforms' tab in the primary links menu.

Execute a new Verify task on this platform via the 'Verify' button in the task list.

Aegir will now re-verify the platform. In the process of this, it will discover your new site and spawn a new 'Import' task for the site.

Once it's done this (it may take a couple of cron runs to dispatch these tasks in the queue) the tasks should turn green.

At this point, presuming you have updated DNS or are overriding DNS via an entry in your /etc/hosts file to access the site via the new Aegir server, you can visit the site in a browser and check around to see that it has worked. Pay particular attention to any links in node content that pointed to paths referencing /sites/community.aegirproject.org if your site was not part of a multisite setup. (Drupal has a habit of storing these paths in the database, or they may have been hard-coded into nodes by users).

Aegir will not go through all your database and update all URLs, so some images or links may be broken. This will need to be done manually. Aegir does attempt to update paths stored in tables such as 'files' though.

It's a good idea to clear the caches, and you may need to get imagecache to rebuild its thumbnails if you use it.