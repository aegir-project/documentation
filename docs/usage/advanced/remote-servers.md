Multiple Servers
================

[TOC]

Aegir supports multiple 'server' entities. These servers have 'services' such as 'Web' or 'Database', and 'service types' which are implementations of that service, such as 'Apache' or 'MySQL'.

Some previous experience in the configuration of Apache and MySQL will help if you want to use remote servers with Aegir. If you haven't had any experience in setting up and maintaining Apache or MySQL before you might want get familiar with the basic concepts first.


Remote servers
--------------

### System dependencies

On the remote server, install these packages

    apt-get install rsync apache2 php5 php5-cli php5-mysql postfix mysql-client sudo

### Aegir user

Any number of remote web servers may be configured. The remote server needs an aegir user created on the system.

    adduser --system --group --home /var/aegir aegir
    adduser aegir www-data    #make aegir a user of group www-data

### Web server configuration

You'll also need to prepare the web server in the same way you did for the master Aegir server during installation:

**For Apache 2.4 and up:**

    a2enmod rewrite
    ln -s /var/aegir/config/apache.conf /etc/apache2/conf-enabled/aegir.conf    

**For Apache 2.3 and lower:**

    a2enmod rewrite
    ln -s /var/aegir/config/apache.conf /etc/apache2/conf.d/aegir.conf 

In case of doubt, just check in /etc/apache2/ whether you have conf.d or conf-enabled, you should only have one of them.

**Don't restart Apache even when it prompts you.** This will be done by the Verify task you'll spawn for the server from the Aegir frontend later.

#### Sudoers

Add the aegir user to sudoers so it can restart Apache

    sudo visudo -f /etc/sudoers.d/aegir

    aegir ALL=NOPASSWD: /usr/sbin/apache2ctl

### Login shell

The remote aegir user will also need a login shell, which can be modified using the chsh command.

    chsh -s /bin/sh aegir

### MySQL access

Aegir connects directly to the remote server's MySQL to create databases and users. Therefore, you need to log into MySQL in the remote server as root and create a user with the following command:

    mysql> GRANT ALL PRIVILEGES ON *.* TO root@aegir_server_ip IDENTIFIED BY 'password' WITH GRANT OPTION;
    mysql> FLUSH PRIVILEGES;

Where

* 'root' is the username on the Aegir server that will be connecting. Leave 'root';
* 'aegir_server_ip' is the IP address of your Aegir server. For example '123.123.123.123';
* 'password' is the password that will be used by your Aegir 'root' user to connect to the remote server's MySQL to create databases and users. It is suggested to store that password in a secure location. To increase security, it is recommended to choose a password with 6 to 16 alphanumeric and or punctuation characters.

If MySQL returns something like "Query OK", that means the command was successful. For example:

    Query OK, 0 rows affected (0.00 sec)

Note the comment about "bind-address = 127.0.0.1" within section 3.6 of the [manual configuration page](install/#36-database-configuration).

When the server is being verified, Aegir will attempt to create a database and grant privileges to a user for that database. If any of these two fails, [verify that your MySQL configuration is correct](http://www.ghacks.net/2009/12/27/allow-remote-connections-to-your-mysql-server/) and that there is no firewall blocking your MySQL port.

You can confirm that MySQL is configured correctly on the remote server by manually connecting from the command line on your Aegir machine:

    $ mysql -h <mysql_server_IP> -u root -p

If your MySql database is running on the same server as the Aegir master and is configured with the name "localhost", you may run an issue where the remote web server tries to access MySql via a local socket rather than the hostname. To resolve this, just change the server hostname for the MySql server in Aegir from "localhost" to the hostname of the server.

### SSH keys

SSH public/private keys should be set up so the main Aegir server's aegir user can access remote web aegir users with no passwords required.

Example:

    $ ssh-keygen -t rsa

Follow the promts. Do not use a passphrase when you create your key. Simply press Enter to leave the passphrase blank. Otherwise, SSH will insist that you enter the passphrase everytime you try to SSH using the key. We don't want SSH to present any prompts.

Put the public key's contents into /var/aegir/.ssh/authorized_keys on the remote web server. The easiest way to do this is to use the ssh-copy-id command.

You should manually login for the first time from your Aegir master server to your remote server as the aegir user, so that the remote web server is added to the known_hosts file in /var/aegir/.ssh on your Aegir master server. Verifying the remote webserver will fail until this has been done.

You can confirm that SSH is set up correctly by manually connecting from your aegirmaster server to your remote server as aegir@ user to aegir@ user:

    [aegir@aegirmaster:~]$ ssh aegir@<remoteserver>

This should directly log you into the remote shell command line without any additional keys pressed.

    [...]
    [aegir@<remoteserver>:~]$

There are many, many tutorials online for setting up ssh keys, and various mistakes can be made by inexperienced users such as permissions etc. Aegir isn't a 'Linux beginner's practice tool', so setting these up is really out of the scope of this document and users are encouraged to research this on their own. (For Ubuntu, [see this article](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)).

If you have followed the instructions above, and your SSH connection gets closed immediately after you try to connect to the remote server as the aegir user, you may not have given the aegir user a shell correctly. Check the /etc/passwd file on the remote server to make sure that the aegir user has a shell.

### Verify the server

Now you can add a new server node in Aegir, set the hostname and/or IP and set the service type to be 'Apache' (or Apache_SSL if this site is to handle SSL sites).

![Create Server Aegir Admin Screen](/_images/create-remote-server.png)

A verify task will be spawned and added to the Task queue ready for dispatching. During a server verification task, various configurations will be set on the Aegir master server and also synced to the remote web server, restarting Apache.

Now when you add a new Platform node in Aegir, you have the option of setting which web server to host it on. If you are not using a makefile but are downloading a platform manually, you must still do this on the main Aegir server. The contents will then be synced across to the web server.

You don't choose a web server when installing a new site. Because a site depends on a platform, its web server is implied by which platform has been chosen. In other words, all sites on a platform are hosted on the same server. You cannot host two sites on the same platform on different servers.

#### Shell commands as root

These are the commands outlined above consolidated.

    apt-get install mysql-client
    adduser --system --group --home /var/aegir aegir
    adduser aegir www-data    #make aegir a user of group www-data
    chsh -s /bin/sh aegir
    apt-get install rsync apache2 php5 php5-cli php5-mysql
    mkdir /var/aegir/.ssh
    cat > /var/aegir/.ssh/authorized_keys <<EOF
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQAB...UF aegir@filer01
    EOF
    chown aegir:aegir /var/aegir/.ssh -R
    chmod 750 /var/aegir/.ssh
    chmod 640 /var/aegir/.ssh/authorized_keys
    a2enmod rewrite
    ln -s /var/aegir/config/apache.conf /etc/apache2/conf.d/aegir.conf
    cat > /etc/sudoers.d/aegir <<EOF
    Defaults:aegir  !requiretty
    aegir ALL=NOPASSWD: /usr/sbin/apache2ctl
    EOF
    chmod 440 /etc/sudoers.d/aegir


Clustering
----------

If you wish to run the same website concurrently on multiple hosts you can use the Pack or Cluster module.

The [**Web Cluster**](#using-the-cluster-module) module uses Rsync to get all files to all servers.

The [**Web Pack**](#using-the-pack-module) module shares files to all servers via an NFS mount.  The NFS setup must be done manually.


### Using the Pack module

If you wish to run the same website concurrently on multiple hosts you use the Pack module. Enable the pack module as you would any other drupal module.

A Pack consists of a single server node that is specified as "pack" and any number of server nodes specified as "apache" or "nginx". Aegir rsync's configuration (Apache & Nginx) to "all" nodes in the Pack and rysnc's the platform and site code to only the "master" node in the Pack. All other "slave" nodes will see the platform and site code via NFS.

#### Configuring the Pack server node:

The "pack" server node will not be used by Aegir to physically deploy sites or platforms on, so consider it more of a 'virtual server group' for logistical purposes only. When creating the pack node, you do not need to supply a valid Server hostname or IP address, so choose a naming convention that makes sense in your environment.

Next, select the "pack" radio button option under the web configuration when creating or editing the server node, in order to designate this server node as the "pack" server.

Now select the "Master" server from the list of Master servers. The Master server will be the server node that Aegir rsync's the platform and site to. Typically, you would choose the Aegir server itself as the 'master'.

Finally, select the "Slave" servers from the list of Slave servers. The Slave servers will have the Apache or Nginx config rsync'd to them but not the platform or site code, since those will be available to all Slave servers via the NFS share.

#### Configuring the Web server nodes:

Configure all web server nodes using [these instructions](#remote-servers). Take care to ensure the 'aegir' user and group on your NFS client machines, have the same UID/GID as that of the NFS server, or else you may run into permissions issues with NFS.

Then mount the files on the remote server.

On the NFS server:

    sudo apt-get install nfs-kernel-server
    echo "/var/aegir/platforms    10.0.0.0/24(rw,no_subtree_check)" >> /etc/exports
    sudo service nfs-kernel-server reload

(Replace the subnet here or add specific /32 hosts as necessary for your environment)

Then on the web servers (Master, if it wasn't also the NFS server, and the Slaves):

    sudo apt-get install nfs-client
    sudo mount 10.0.0.1:/var/aegir/platforms /var/aegir/platforms

(Replace the NFS server's IP here with that of your master server/Aegir server)

Add this to your fstab on the servers that mount the NFS share, so that the share is mounted on boot:

    10.0.0.1:/var/aegir/platforms /var/aegir/platforms nfs rw 0 0

##### Creating a Platform on a Pack:

When configuring a Platform to be deployed on a Pack, choose the "Pack" server node from the Web server radio group during the Platform node creation. Then you choose this Platform as usual when adding a site, and that site will be served from any servers within the Pack.

##### Caveats

Relying on an NFS share to serve your entire /var/aegir/platforms can be a single point of failure if the NFS share becomes unavailable. You may want to look into providing some sort of failover for NFS (google for things like DRBD and Heartbeat), or using some other form of redundancy for your NFS (Netapp filer clusters etc)

Below is an example diagram of a Pack cluster known to be functioning in production (with an optional MySQL-MMM cluster out of scope for this documentation), which may help you visualise the Pack and how it can be put together.

![Diagram of Pack configuration with multiple servers](/_images/aegir-pack.png)


### Using the Cluster module

This documentation page is about the **Web Cluster** module.

#### Setting up Web Clusters

In this setup, we will add two web servers and a single database server.

1. Under **Admin > Hosting > Features**, enable the "Web Clusters" module.
2. Setup two or more additional **Remote Servers** using the [Remote Server Instructions](#remote-servers).
3. Create Remote Server Nodes: Visit **Servers > Add Server** and enter the hostname of your remote server.  For **Web Server**, select *Apache*, *Apache SSL*, *NGINX*, or *NGINX SSL*.
4. Repeat Step 3 for all of your web servers.
5. Create Cluster Server Node: Visit **Servers > Add Server** and enter any hostname (something like "cluster" so you can identify it.).  For **Web Server**, select *Cluster* and select all of the web servers you wish to add to the cluster.
6. Setup Database Server: You have two options here.
  1. Provision a new, separate database server using only the [MySQL section](#mysql-access) above.  Then visit **Servers > Add Server**, enter the new server's hostname, and select "MySQL" for "Database" and enter the root username and password you set during the Remote Server Instructions.
  2. Use an existing server other than "localhost". Aegir installs a database server called "localhost" by default. You cannot use this server for sites installed on remote servesr.  Find the default web server aegir installed (named after the hostmaster hostname).  Click *Edit*, then select "MySQL" for "Database" and enter the root username and password.  If you use the server with the same hostname as the "localhost" database server, you can use the same root username and password.
7. Make sure the Database Server Hostname resolves to the database server IP.  This can be done with DNS or by manually editing the /etc/hosts file on the remote servers.
8. Create or Edit a platform node, and select the cluster server you created in step 5.
9. Create a site node, and select the database server you created in step 6.

If all goes well, the site should install.

To test that the site is available from both servers, you must edit your DNS or hosts file:

1. Given your site node URL is myclustersite.com, edit /etc/hosts or your DNS so myclustersite.com resolves to the IP of remote server #1.
2. Visit the URL http://myclustersite.com in the browser. If the site works, we know we can load the site from remote server #1.
3. Edit /etc/hosts or your DNS so myclustersite.com resolves to the IP of server #2.
4. Visit the URL http://myclustersite.com again. If the site works, we know we can load the site from remote server #2.

#### Final Tasks

There are two important final tasks to get a working drupal site running on a server cluster:

1. Load Balancer.
  Now you must setup a load balancer that will route requests to both remote server #1 and #2.

2. Shared Files folder.
  Uploaded files will only go to one server when you upload them.  They will appear as missing when requests come in to the other servers.

  There are a number of ways to deal with this: You can use NFS to create a shared files folder, or you can configure your load balancer to always send requests for the files folder to a single server.

  Setting up a load balancer and dealing with the files folder is out of scope of this article, for now.


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
