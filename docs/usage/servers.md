Servers
=======

[TOC]

Aegir supports multiple 'server' entities. These servers have 'services' such as 'Web' or 'Database', and 'service types' which are implementations of that service, such as 'Apache' or 'MySQL'.

We designate the server on which Aegir itself is running as the "master" server. Any other server is thus considered "remote". [Remote servers](/usage/servers/remote-servers/) can be combined into [clusters](/usage/servers/clustering/) for high-performance and high-availability scenarios.


Platforms on remote web servers
-------------------------------

If you have been using Aegir for a while simply because it handles updates, moving to new releases, cloning, aliases and everything Drupal with such elegance on a single server - you might not be entirely clear on what changes when you make the move to multiple servers. Hopefully this will help.

### Understanding the process

Multiple servers are great even if you do not have to manage a server farm. Having a server for development that you are free to break, one for testing that you hope will not break, one for approvals so your clients can go to see how their "real" website will look, and one for production that serves the daily grind of live sites is a wonderful thing. This DTAP (Development, Testing, Acceptance, and Production) structure works and it is surprisingly easy to accomplish under Aegir.

If you are managing a farm, creating massive numbers of servers is quick and easy with Aegir - especially if you use virtual machines like Xen where a server image can be cloned or generated using LVM disks that can be migrated (even while running) to other physical machines.

When it is time break out an Aegir master controller and remote slaves you have to remember that Aegir is the place that it (almost) all happens. Everything is stored in Aegir - including your remote platforms and your sites - and then distributed to the slave servers by Aegir in the Verify process.

Remember too that sites reside on platforms, and platforms reside on servers. You are turning your Aegir install into an Aegir Master as the hub, and the Aeger Remote Slaves as the spokes. To move into this "hub and spoke" architecture, you must build the remote server, tell the Aegir Master about that server and verify it, install a Drupal platform for that server on the Aegir Master and finally tell the Aegir Master about that Drupal install and verify that. You then create the sites on the platform you want based upon the install profile you select for that site.

### The mechanics

1. Build a server image on your network to become your Aegir Slave. This can be anything from a separate headless server to a Virtual Machine image on a physical machine with a dozen other Virtual Machines on it. As long as it has its own IP and is capable of running a Drupal image, it will work.

2. Ensure that your new server image is resolved in your DNS by adding it to your zone files and incrementing the serial number in the zone file. Aegir can manage that for you now to some extent, but you might want to wait until that feature leaves the experimental status. Make sure the entries in /etc/hostname and /etc/hosts all match your zone files (including reverse DNS). Reload/restart the DNS server. Any DNS slaves should automatically get notified and the incremented serial number should make sure they reload automatically. Test the resolution and ping the new Aegir Slave server image from the Aegir Master to Ensure that the IPs all resolved properly.

3. Follow the steps for [Setting up a remote web server](remote-servers.md) carefully. You should be able to use cut and paste if you are using Debian Stable or the like. In any case, you will need those packages. You do not need Aegir on the slave, but you do need the aegir user and that user must have a shell. I prefer the bash shell, but the sh shell in the instructions will do.

4. You need to have the "passphraseless" ssh working and proven before you continue. A web search on the terms "passphraseless ssh" plus your operating system will generally get you great instructions, but putting those instructions here for every possible combination is beyond the scope of this example. In terms of helpful hints, however, you do not need to have a password for the aegir user on the Aegir Slave. Your access will be this certificate. By simply hitting enter when it asks for the passphrase, you make the certificate passphraseless. You should carefully set the permissions and back these certificates up to a secure location. You MUST log in manually from the Aegir Master to the Aegir Slave the first time or your verify will fail. This is because the system has to add the ssh keys to the known hosts and it will ask about about it before it will allow the connection. Aegir nows nothing about this and will not answer, so the verify will fail. To be absolutely sure it works, after exiting the initial login, you should retry the login to verify it does not ask for a password and then exit the login and try an rsync (which is the process by which the Aegir Master controls the Aegir Slaves). All of these commands should work from the Aegir Master to the new Aegir Slave while logged into the Aegir Master as the aegir user with no request for a password, proving that the systems are exchanging keys (do not proceed until you can do this flawlessly without having to enter a response):

        ## Start from the Aegir Master as the aegir user
	    # Log in to the slave from the Aegir Master and exit without any password or reply required
	    ssh slavename.example.com
	    exit
		# Create a test file for rsync
		echo 'rsync test file' > MyRsyncTest
		# Send the test file to the remote slave by name using rsync
		rsync -azvv /var/aegir/MyRsyncTest aegir@slavename.example.com:/var/aegir
		# Log back into the remote slave
		ssh slavename.example.com
		# Make sure the file is there and readable
		cat /var/aegir/MyRsyncTest
		# Delete the test file from the remote slave
		rm /var/aegir/MyRsyncTest
		# Exit the ssh login session on the remote
		exit
		# Delete the test file on the Aegir Master
		rm /var/aegir/MyRsyncTest

5. Go to Add Content/Server and put in the server name, leave the Database set as None (Your sites will use the Aegir Master database and none is required on the Remote Slave) and under Web select apache or apache ssl (to match the example) then click Save. After the process completes you should see that it found the IP address properly on its own, plugged it in all by itself and then Verify quite nicely. If you look on the Aegir Slave remote server you will see that you now have directories named config and platform.

6. Now that hard decision for platform choice comes in play. My preference is a Drupal install that is the same release as the Aegir Master server or higher. Since sites are eventually destined to be Drupal 8 one day, and the Aegir system is not Drupal 8 ready as of this writing but heading that direction, making at least one Drupal 8 slave is a good idea so that you can migrate sites to that platform as they are ready. If you are migrating sites off of your existing Aegir and onto slaves, migrating to platforms of the same Drupal version makes that process much easier. In any case, firm up your decision about which Drupal version you want to install on your new Aegir Slave Platform.

7. Time to create that platform. The big thing here is to do it right the first time and make certain that the name of the platform is unique to any platform on your Aegir Master server. If you look in your /var/aegir directory you will notice that Aegir has its own naming convention. For instance, you should see /var/aegir/hostmaster-6.x-2.4 as the platform for the Aegir 6.x-2.4 release. You need to consider doing the same. For instance, name your platform something like: /var/aegir/slavename-drupal-version and you will make it easier to distinguish which server the platform resides upon and the drupal version so that you can manage updates and migrations more easily. Make certain that you are not destroying one platform when creating another. Be aware that at some point Aegir started organizing platforms into a separate directory: /var/aegir/platforms and you should change to that structure if you have an older original install. Again, be careful you are not overwriting an existing install when you create the new one. For instance, if you already have a Drupal install in /var/aegir/platforms/drupal-7.38 the code below will attempt to recreate it on top of the existing install. If you are using a Makefile, be sure that the same problem does not occur in that makefile. To create a Drupal 7 Aegir Remote Slave Platform, log into the Aegir Master as the aegir user and run this simple code (of course edit it to match the platform and name you want to create:

		# Make sure you are in the proper directory
		cd /var/aegir/platforms
		# Install Drupal 7.38
		drush/drush.php dl drupal-7.38
		# Rename the platform to your own naming convention
		mv /var/aegir/platforms/drupal-7.38 /var/aegir/platforms/slavename-drupal-7.38

8. The final step in creating this platform is to actually tell the Aegir Master about it so that it can distribute it (via rsync) to the new Aegir Remote Slave. Simple enough. Just go to Create Content/Platform and the Create Platform screen comes up. Under Name write something descriptive and unique. In the example so far, this will be something like 'slavename Drupal 7.14' or in some way adhere to your chosen naming convention. I use the directory name I created above since that serves two purposes. Under Publish Path you will put the path that holds this Drupal distribution. In the example above that will be /var/aegir/platforms/slavename-drupal-7.14 but the path will be reflected from the Aegir Master to the Aegir Slave and will be the same on both. If you are using a Makefile you need to enter the full path and name of that Makefile here as well, but the creation and use of the Makefile is beyond the scope of this example. Lastly choose the server that this platform will reside upon. When you are finished, double check the screen and then click on Save. The Verify step should execute flawlessly and when it does, your Aegir Remote Slave will be complete and ready to accept sites.

9. Lather, rinse and repeat for each Aegir Remote Slave.

### Creating sites on the remote

The only difference you will notice when creating a new site under Aegir will be the choice of platform. The Create Site screen is quite self explanatory, but the main thing is to select the Install Profile that you require and then the list of Platforms that support that install profile will be populated.

As usual, Aegir makes it all faster and easier than you expect.
