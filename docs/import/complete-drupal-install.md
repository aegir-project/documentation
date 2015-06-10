Importing a complete Drupal platform
====================================

Table of Contents

1. Transfer the codebase
2. Transfer the Database
3. Setup Platform in Aegir
4. Import and Verify Sites In Aegir
5. Migrate To Another Platform
6. Troubleshooting

This way of importing sites is often considered the safest whereby you transfer across an entire codebase containing Drupal core, and define it as a Platform first. This makes sure you bring any dependencies, contrib modules etc, in exactly the same version and configuration that is referenced in the database for the site.

1. Transfer the codebase

	Create a tarball of the codebase on the old server. Within the drupal root directory type: tar -zcvf example.com.tar.gz * .htaccess

	Note that we have to specify including the .htaccess because it isn't picked up by the * wildcard due to being prefixed with a full stop. It is important to bring along the .htaccess for a platform so that Aegir can consider these settings when it generates Apache configuration for this platform.

	Then, on the new server you can create a directory for it in the codebase you want to put it under:

		cd /var/aegir/platforms
		mkdir your-new-platform
		cd your-new-platform
		wget http://example.com/example.com.tar.gz
		tar -zxvf example.com.tar.gz

	Instead of using wget you could of course use SCP or FTP to transfer the tarball onto the Aegir server.

	Once the file is unpacked, check the ownership and group of each file by using ls -al. It should be either 'aegir:aegir' or 'aegir:www-data'. Change it if necessary. We have to assume you have basic knowledge of Linux and POSIX permissions - we can't be responsible for teaching you sysadmin skills :)

	In particular, files that may have been uploaded on your site via modules like imagecache, upload, Profile pics etc, may need to have their ownership changed: sudo chown -R aegir $platform/sites/*/files

	If your site previously resided in sites/defau1t you need to move it because Aegir ignores the default directory (it has no way of understanding what URL this would be accessed by, so it is impossible to 'manage' it). Each site needs its own directory with the correct domain in typical Drupal multisite design:

		mv sites/defau1t sites/example.com

	If you do this, you may need to also update your filepath's stored in the files table by running the following query on your database:

		UPDATE files SET filepath = REPLACE(filepath, 'sites/defau1t', 'sites/example.com');

		There are many other places in the database that can suffer the same problem, the node_revisions table for example. If you have phpMyAdmin available you can easily search the entire database for 'sites/defau1t'. Select the database, then click the Search tab. Once you know which tables are affected you can run a variation of the above query to correct these records.

2. Transfer the Database

	Note: This step is not necessary if you are moving your site from one directory on your server (e.g., /var/www/html) to a newly created Aegir directory on the same server (e.g., /var/aegir).

	Make a backup of the database on your old server Backup and Migrate module is good for this, or you can use phpmyadmin or your favourite MySQL management tool. It's a good idea to truncate the cache, search and watchdog tables first to reduce the size of the database (Backup and Migrate does this for you).

	Aegir does not provide support for importing databases with prefixed tables so it is best to remove prefixes form the database. However, a [workaround is available](http://drupal.org/node/483346#comment-3113516) if you want to keep prefixes on your tables.

	Unlike traditional Drupal projects the database settings cannot be found in settings.php. The database credentials are stored in the Apache vhost config of the associated site. This is a security measure implemented by the Aegir project.

	In the vhost directory on your old server, /var/aegir/config/server_master/apache/vhost.d/, there is a vhost file for every site on your server. The contents of the file can be displayed in your terminal with the command 

		cat thenameofthefile

	At the top of the file you will find the credentials needed to create a database on the new server.

	On your new server, manually create a new database and upload the .sql file from your backup.

	Then create the mysql user that your site accesses the database as, and grant it all permissions on that database except 'GRANT'.

	Connect to the database on the old server: 

		mysql -u root -p

	Create the new database: 

		create database yourdatabasename

	Add the user: 

		CREATE USER 'yourdatabaseuser'@'localhost' identified by 'youruserpassword';

	Give the necessary privileges: 

		GRANT ALL PRIVILEGES ON yourdatabasename.* TO 'yourdatabaseuser'@'localhost';

	Import the database: 
		mysql -u yourdatabaseuser -p -h localhost yourdatabasename < thenameofthedatabase.sql
	
3. Setup Platform in Aegir

	In the Aegir web interface click on 'Create Content' and 'Platform'. This step is often easiest when using the admin-menu on the top of your screen.

	Enter the name you want to use for the platform (eg 'My Platform import'), and the path to the platform on the server (eg '/var/aegir/platforms/your-new-platform')

	Click Save to submit the form, and Aegir will spawn a Verify task for this platform into the queue.

	Once the task is complete this the task should turn green (click through to the homepage to refesh and check. If not, see 'Troubleshooting' below).

4. Import and Verify Sites In Aegir

	Upon completion of the above Platform verification task, Aegir will discover your new site and spawn a new 'Import' task for the site.

	Once it's done this (it may take a couple of cron runs to dispatch these tasks in the queue) the tasks should turn green (if not, see 'Troubleshooting' below).

	At this point, presuming you have updated DNS or are overriding DNS via an entry in your /etc/hosts file to access the site via the new Aegir server, you can visit site in a browser and check around to see that it has worked. Pay particular attention to any links in node content that pointed to paths referencing /sites/community.aegirproject.org if your site was not part of a multisite setup. (Drupal has a habit of storing these paths in the database, or they may have been hard-coded into nodes by users).

	It's a good idea to clear the caches, and you may need to get imagecache to rebuild its thumbnails if you use it.

5. Migrate To Another Platform

	Now that you have your sites under Aegir's control you can take advantage of its power, and easily migrate them to another platform. In the event that your sites were on old versions of Drupal core and a new one is available and present as another Platform on your Aegir server, you can use the Migrate process to upgrade those sites onto the new core.

	For details on how to Migrate sites, consult the [Migrate documentation](@todo).