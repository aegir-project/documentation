Installation Guide
==================

[TOC]

Aegir requires some special permissions to your server in order to automate
some configurations. For example, when a new site is installed, the web server
will be automatically configured (vhost) and restarted. Therefore Aegir cannot
be installed on a shared hosting environment. Consult the [system
requirements](#system-requirements) to ensure your server(s) are up to the task.

System Requirements
-------------------

A system capable of running Drupal

The Aegir system is entirely Drupal based, and has the same base requirements that Drupal does (with the exception that it won't run on Windows). See more notes on Unix and LAMP/LEMP requirements below.

### Your own server

The low level of access required to be able to configure and run this system is very far beyond what is commonly available to users with shared hosting. A VPS from any popular provider such as Linode, Rackspace, Slicehost, Amazon EC, etc. will do fine. You will need root access to the server and the server needs to be dedicated to Aegir.

### A Unix-based operating system

Aegir must run on some flavour of UNIX, because the majority of functionality in this system occurs in the back-end, through command line scripting. There are also several features (such as symlinks), that are not available to users on Windows. There are no plans currently to add Windows support. See the operating system support page for more information.

### Web server

You will need at least one dedicated web server, running Apache. We generally work with Apache 2. Aegir also supports the Nginx web server, but requires at least version 0.7.66 or newer. Since Nginx doesn't provide php-cgi or php-fpm (recommended) modules, you will need to install and run php-fpm server separately. You can find useful examples and tips in the third party Barracuda installer available at the barracuda project page.
N.B.: This third party installer is not supported by the core Aegir developers, but you can find helpful community support at the boa group.

See also [installing with Nginx](#install-with-nginx).

### PHP

Aegir depends on Drush 6.5.0 or later, which requires PHP 5.3 or higher. You also need to have the command-line version of PHP to run Drush properly, and the MySQL extensions for PHP.

### Database server

You will require a database server, obviously. Aegir currently only supports MySQL and MariaDB. It is preferable to use a dedicated (not shared-hosting) server since Aegir will create database users and will require the use of a MySQL root user.

### Mail transfer agent

Aegir requires an MTA (Mail Transfer Agent) installed on your webserver in order to be able to install new sites to your new platform. If you don't have an MTA, the site installation will fail with message like "could not send email". Additional messages will show that site has been removed because of this problem. To remedy the situation simply install an MTA like sendmail, postfix, or exim and do the minimal configuration.

### A Fully Qualified Domain Name (FQDN)

  For example, `aegir.example.com`. The hostname returned by the commands ``hostname -f`` and ``uname -a`` must resolve to the IP address of your server. After setting up your FQDN you must restart your server with a ``reboot`` command so your changes take effect. Note that newly created domain name usually take 24 to 48 hours to fully start working. This period, called propagation, is the projected length of time it takes for root name servers and cache records across the entire web to be updated with your website's DNS information.


### Other utilities: sudo, rsync, git and unzip

Aegir installs itself via a Drush Make makefile that downloads via git if you want the bleeding edge code, or via wget if you want the latest official release. If you want the latest development version, and don't have the git program you will need to install it on the server.
The jQueryUI library is used in the Aegir UI, unzip is required to extract it. Sudo is required to allow the aegir user the limited privilege to restart the webserver when required. Rsync is used to sync files to remote servers.

### No conflicting Control Panels

Other popular control panels such as Plesk, cPanel etc, are designed to manage all aspects of Apache configuration and other areas that Aegir also is intended to be used for.
Running Aegir alongside such control panels is not supported and very likely may cause you problems or difficulties installing or running Aegir. Filing bug reports that are caused by interference by another control panel will likely be closed unless the problem can be fixed without causing problems for other Aegir users. Proceed at your own risk!

### Requirements of Drupal distributions

Some Drupal distributions, such as OpenAtrium, are specialized products that may contain unique prerequisites for optimal performance. Such examples may include raising the php-cli program's memory_limit to something higher than 64M.
Please note that this is not a requirement of Aegir but of the distribution you are trying to install a site on. Thus the Aegir documentation may not officially 'require' such performance settings, but be aware that Aegir may report errors if the system was under-resourced to complete such a task.


Debian/Ubuntu
-------------

These are the installation instructions that are recommended on Debian. Aegir dependencies (Apache, MySQL, PHP...) are also automatically installed.

If you wish to install Debian packages over an existing manual install, it's possible. See [the Debian upgrade procedures](/install/upgrade/#upgrades-with-debian-packages).

Debian packages are uploaded to http://debian.aegirproject.org/ shortly after a release. We eventually want to upload those packages to the official archives, but this will take some adaptation and time to sponsor the packages in.

### 1. Ensure requirements are satisfied

Here's what you'll need to install Aegir on a Debian or Ubuntu system:

* Basic Linux system administration skills
* Root access to your server
* An up-to-date system and applications (`sudo aptitude update && sudo aptitude safe-upgrade`)

See [system-requirements](#system-requirements)

### 2. Add project repositories

Use this command to add the Aegir package "Software Source" repository to your system:

    echo "deb http://debian.aegirproject.org stable main" | sudo tee -a /etc/apt/sources.list.d/aegir-stable.list

To install a customized Debian package, see the [developer instructions for the debian package](/community/release-process/debian-packaging/). Other distributions are available for courageous people that want to try development versions.

**dev note**: to install the development version of Aegir, you can use the `unstable` or `testing` distribution above.

### 3. Add archive key to your keyring

This repository self-signs packages uploaded to it (and packages uploaded are verified against a whitelist of trusted uploaders) using OpenPGP (GnuPG, to be more precise).

Use these commands to download and add the repository's PGP key, then update the package list on your system:

    wget -q http://debian.aegirproject.org/key.asc -O- | sudo apt-key add -
    sudo apt-get update

### 4. DNS configuration

Aegir requires a properly configured "FQDN" (Fully Qualified Domain Name) be assigned to the machine. In practice, this means that the hostname returned by the ``hostname -f`` and ``uname -n`` shell commands should resolve to the IP address for this server, and vice versa, with the ``resolveip`` command (included with the mysql-server package).

For Ubuntu, /etc/hosts should have entries that look like:

    ::1 host.example.com host ip6-localhost ip6-loopback
    127.0.0.1 host.example.com host localhost
    123.123.123.123 host.example.com host localhost

To set this up in a virtual machine (e.g. Virtualbox), here are the steps:

* Create a new VM
* Go to settings->network.  Enable Adapter 2, and set to "host-only"
* Install Ubuntu.  Set hostname as FQDN during install
* You may need to add the lines `auto eth1` and `iface eth1 inet dhcp` to /etc/network/interfaces

If you have a virtual machine already setup and want to change the FDQN:

* change /etc/hostname using: ``sudo hostname NEW_NAME``
* change /etc/hosts using: ``sudo nano /etc/hosts`` and change name
* reboot and test ``hostname -f``, ``uname -n``, ``resolveip NEW_NAME``, ``resolveip IP``
* [YMMV - Your Mileage May Vary](http://www.ducea.com/2006/08/07/how-to-change-the-hostname-of-a-linux-system/)

### 5. (Optional) Manual installation of MySQL

Please note that Ubuntu 12.04 LTS installs, by default, an insecure MySQL installation that contains an anonymous user grant, allowing anyone to login without a password. This breaks Aegir functionality.

If you are running Ubuntu 12.04, you should install MySQL manually, and then ensure it is installed securely:

    $ sudo apt-get install mysql-server
    $ sudo mysql_secure_installation

When running 'sudo mysql_secure_installation', answer 'Y' to 'Remove anonymous users?'

    By default, a MySQL installation has an anonymous user, allowing anyone
    to log into MySQL without having to have a user account created for
    them.  This is intended only for testing, and to make the installation
    go a bit smoother.  You should remove them before moving into a
    production environment.

    Remove anonymous users? [Y/n] Y
     ... Success!

Now you can proceed with installing Aegir below.

### 6. (Optional) Install with Nginx

If you want to use Nginx instead of Apache, you'll have to explicitely tell apt-get:

    $ sudo apt-get install aegir3 nginx php5-fpm

This is because the package expects Apache2 by default.

### 7. Install Aegir

To install Aegir version 3, frontend and backend, use the following command:

    $ sudo apt-get install aegir3

This will prompt you for the required information (MySQL password, Postfix configuration...) and go ahead with the install.

During the Postfix configuration, the following options appear: "No configuration, Internet site, Internet with smarthost, Satellite system, Local only". That first text screen only allows to use the tab key to select "OK", and then the enter key to display a second screen where you can select one of the choices. The default is "Internet site", useful in most cases to enable the server to send email messages, for example to the admin.

At the end of the installation, you will receive an email message or, if the user "aegir" has been assigned with a local email account during the installation, the file /var/mail/aegir will contain the message. It will include a one-time login to your new Aegir control panel, that is a URL to copy into your browser so that you can set the password for the "admin" user.

#### 7.1 Custom options

The Debian packages allow for a number of options to be set for the various packages:

* [aegir3-provision](http://cgit.drupalcode.org/provision/plain/debian/aegir3-provision.templates)
* [aegir3-hostmaster](http://cgit.drupalcode.org/provision/plain/debian/aegir3-hostmaster.templates)
* [aegir-cluster-slave](http://cgit.drupalcode.org/provision/plain/debian/aegir3-cluster-slave.templates)

For example, to create the front-end hostmaster platform as working copies (Git checkouts for all modules) you can set the `aegir/working-copy` option thusly (noting that `debconf-set-selections` is part of the [debconf-utils](https://packages.debian.org/search?keywords=debconf-utils) package) before installing Aegir:

    $ sudo apt install debconf-utils
    $ mkdir -p ~/projects/aegir/core
    $ git clone https://git.drupal.org/project/provision.git ~/projects/aegir/core/provision
    $ echo "aegir3-hostmaster aegir/makefile string /home/$USER/projects/aegir/core/provision/aegir-dev.make" | sudo debconf-set-selections
    $ echo "aegir3-hostmaster aegir/working-copy boolean true" | sudo debconf-set-selections
    $ sudo apt install aegir3

Note that this is not currently possible for the Provision back-end, but there is [a feature request](https://www.drupal.org/node/2830610) for it.  For the moment, you'll have to manually replace it with a link to the above Git clone after installation:

    $ sudo mv /usr/share/drush/commands/provision /tmp/provision.default
    $ sudo ln -s ~/projects/aegir/core/provision /usr/share/drush/commands

##### 7.1.1 Custom make files

If you have your own Drupal make file, you can go ahead with the above process, but change the make file to the one you want:

    $ echo "aegir3-hostmaster aegir/makefile string /var/aegir/makefiles/aegir/aegir-custom.make | sudo debconf-set-selections
    $ sudo apt-get install aegir3

This allows you to specify the makefile path for your custom distribution of Aegir. To maintain these customizations, you'll need to ensure you do the same when upgrading.

An example custom Aegir makefile could look like [this](http://cgit.drupalcode.org/provision/tree/aegir-dev.make).

### 8. Troubleshooting the install

To make the install smoother, the install command is run without much debugging information, which can make diagnostics pretty hard. For this, there's a special environment variable you can set that will trigger debugging output. Install aegir with this:


    $ env DPKG_DEBUG=developer sudo apt-get install aegir

You can build your own Debian packages from our repositories using [these instructions](/community/release-process/debian-packaging/).

Also see the general [Installation Trouble-shooting](/install/#installation-trouble-shooting) section below.

Manual Installation
-------------------

This is the process you need to follow if Aegir doesn't have packages for your distribution. We currently provide `debian` packages and others should be coming, if you help! This manual assumes you are fairly familiar with the UNIX commandline interface, but should be possible to follow through if you copy and paste faithfully **all** steps of the procedure.

A note on supported systems:

These instructions provide example commands for a Debian-like distribution, but should be fairly easy to adapt to other environments. This document is meant as a canonical reference that should work on every supported platform. It can also be used for people porting Aegir to new platforms or installing on alien platform for which Aegir is not yet packaged.

It currently contains special recommendations for CentOS, RHEL 6, Arch Linux and Solaris. Users of those platforms are also strongly encouraged to review the :doc:`common-problems` that occur on those platforms. Aegir is also known to be installable (and was developed partly on) Mac OS X, but that process is so obtuse that it has a [separate page]() for the first part of the manual (up to Install Aegir components).

Installing Aegir may seem daunting at first (which is why we provide automated installs through packages), but once you get around it, it's fairly simple.  Note that these instructions setup a complete Aegir system. If you want to only setup a new remote web/db server, it should be sufficient to install the system requirements (step 1), configure them (step 2) and follow the [Remote server how-to]().

### 1. Review System Requirements

See the [System Requirements](#system-requirements)

### 2. Install system requirements

To install the required components, use your system's package manager. For Debian based systems this would look like:

    $ sudo apt-get install apache2 php5 php5-cli php5-gd php5-mysql php-pear postfix sudo rsync git-core unzip

*Note*: replace `apache2` with `nginx php5-fpm` to install nginx on Ubuntu Precise or newer.

### 3. Configure system requirements

#### 3.1. Create the Aegir user

The provision framework of Aegir requires that the scripts run as a non-root system account, to ensure that it can correctly set the file permissions on the hosted files.

Also to ensure that the file permissions of the hosted sites are always as safe as can be, and especially to make sure that the web server does not have the ability to modify the code of the site, the configured system account needs to be a member of the web server group, in order to be able to correctly set the file permissions.

While you can choose another username, most aegir documentation assumes the Aegir user is ``aegir``, its home directory is ``/var/aegir``
and the webserver group is ``www-data``.

    $ sudo adduser --system --group --home /var/aegir aegir
    $ sudo adduser aegir www-data    #make aegir a user of group www-data

#### 3.2. Webserver configuration

Aegir supports two popular web servers, Apache and Nginx.

##### 3.2.1. Apache configuration

Aegir assumes a few Apache modules are available on the server, and generates its own configuration files. The way we enable this is by symlinking a single file which contains all the configuration necessary. In Debian-based systems, you should symlink this file inside ``/etc/apache2/conf.d`` ior ``/etc/apache2/conf-enabled``  that will be parsed on startup or alternatively you can place include that file in your apache.conf/httpd.conf. We prefer the former. In other systems there are similar ways to accomplish this. Consult your OS's documentation if unsure.

If you are on a Debian-based system, you will also need to enable the mod_rewrite module manually.

Run the following shell commands as root. First, configure Apache to enable RewriteEngine:

     $ a2enmod rewrite

Finally, create a symlink from an apache configuration file to a folder within the /var/aegir/:

     $ ln -s /var/aegir/config/apache.conf /etc/apache2/conf.d/aegir.conf

###### 3.2.1.1. Ubuntu 14.04+

Ubuntu 14.04 departs from Debian and previous Ubuntu Apache setup in that it doesn't use ``/etc/apache2/conf.d`` any more and better separates out ``sites-enabled`` from ``conf-enabled`` configurations. So:

     $ ln -s /var/aegir/config/apache.conf /etc/apache2/conf-available/aegir.conf
     $ a2enconf aegir

Do not reload/restart Apache if prompted to after running these commands, it will fail.

##### 3.2.2. Nginx configuration

(If you just succeeded in installing Apache, please skip this section.)

Aegir assumes standard Nginx configuration is available on the server, and generates its own configuration files. The way we enable this is by symlinking a single file which contains all the configuration necessary. In Debian-based systems, you should symlink this file inside ``/etc/nginx/conf.d`` that will be parsed on startup.

Please make sure your nginx installation is up and running before continuing. On Ubuntu 12.04 Server, for instance, you must edit /etc/nginx/nginx.conf and uncomment the line "types_hash_max_size 2048;" in order for nginx to start successfully.

    $ sudo ln -s /var/aegir/config/nginx.conf /etc/nginx/conf.d/aegir.conf

Do not reload/restart Nginx after running these commands, it will fail.

The installer script creates the configuration file referenced by the newly created symlink.

#### 3.3. PHP configuration

Some complex installation profiles or distributions require a PHP memory limit that is higher than the default. To avoid common errors when installing sites on some distributions, the PHP command line tool should be configured to use 192Mb of RAM.

Change the memory_limit directive in /etc/php5/cli/php.ini to read:

     memory_limit = 192M      ; Maximum amount of memory a script may consume (192MB)

Most modern Drupal sites require around 96M or even 128M of RAM for certain operations. This is far more than what is provided by the default PHP configuration.

Change the memory_limit directive in /etc/php5/apache2/php.ini to read:

    memory_limit = 128M      ; Maximum amount of memory a script may consume (128MB)

If your distributions require more memory than these limits, then use some common sense and update it as appropriate to suit your individual needs.

#### 3.4. Sudo configuration

Next, we need to give the Aegir user permission to execute the command to restart the Web server without entering a password.  Let's create a file to do so.

    $ sudo --preserve-env visudo --file=/etc/sudoers.d/aegir

Add these lines:

    Defaults:aegir  !requiretty
    aegir ALL=NOPASSWD: /usr/sbin/apache2ctl # for Apache
    aegir ALL=NOPASSWD: /etc/init.d/nginx    # for Nginx

The *visudo* command should ensure safe permissions on the file after saving.  If not, change them:

    $ sudo chmod 0440 /etc/sudoers.d/aegir

Note - the path to your apache2ctl program may differ from this example. On some systems it may also be called 'apachectl' instead of apache2ctl. Adjust to suit your own requirements.

#### 3.5. DNS configuration

Aegir requires a properly configured "FQDN" (Fully Qualified Domain Name) be assigned to the machine. In practice, this means that the hostname returned by the ``hostname`` and ``uname -n`` shell commands should resolve to the IP address for this server, and vice versa.

If you only intend to use Aegir on a single server, it is acceptable for the resolved IP address to be the '127.0.0.1' loopback address.

If you intend to manage multiple servers using Aegir, you will need to make sure that the IP address is the public IP of this server.

You can add multiple entries to your /etc/hosts file for testing purposes, for example:

     127.0.0.1 aegir.example.com example.com test1.example.com test2.example.com test3.example.com

Then you can use test1.example.com to create your first site.

#### 3.6. Database configuration

Aegir supports MySQL right now. It is best to install the MySQL server using your Linux distribution's package manager.

    $ sudo apt-get install mysql-server

To make sure that the Aegir backend, and all the possible web servers can reach your database server, you need to configure mysql to listen on all the public IP addresses available to it.

Again, as root, edit the MySQL configuration file `/etc/mysql/my.cnf` configuration line to comment out by placing a # at the beginning of the line as follow:

Before:

    bind-address        = 127.0.0.1

After:

    # bind-address      = 127.0.0.1


Without this line commented out, MySQL will listen only on localhost for database connection requests.

Now you need to restart mysql, to clear any caches.

     $ sudo /etc/init.d/mysql restart

The installer will prompt you for your MySQL root user password. The root user will be used to make administrative tasks such as creating new databases, and granting and revoking access to those databases for sites.

Even though MySQL is now listening on all IP's, it will not allow invalid users to connect to the databases, without the correct user accounts configured.

If you are concerned about MySQL being accessible in this way, you can also configure your firewall to only allow incoming connections from certain addresses. This is outside the scope of this document however.

Note that Aegir will ask you for your MySQL root password. If you do not want to use your regular root password for Aegir, you will need to create another root account for Aegir using a MySQL command like:

     mysql> GRANT ALL PRIVILEGES ON *.* TO 'aegir_root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;

Note: If you are running your Aegir databases on a remote DB server, you will want to create this aegir_root user. The install will often fail if you're trying to use the root user on a remote database. See [this issue]() for details.


##### 3.6.1. OS-specific configurations

NOTE : If you are running either Ubuntu 12.04 LTS, RHEL 6 or Arch Linux, you should still install MySQL in the same way as above.  However, once done, you must now remove the anonymous, passwordless login that those platforms creates by default.

    $ sudo mysql_secure_installation

Otherwise, Aegir will fail to install and work at all. See [this FAQ entry](#installation-trouble-shooting).

### 4. Install Aegir components

Next step is to install the Aegir software components themselves, that is: drush, provision and hostmaster.

#### 4.1. Install drush

Before installing Aegir proper, you first need to install Drush. See then Drush [installation documentation](http://docs.drush.org/en/master/install/)

#### 4.2. Stop! Now become the Aegir user

The remaining of this manual assumes you are running as the Aegir user. Things will go very wrong if you do not change your shell credentials to become that user.

    $ sudo -u aegir -s -H

#### 4.3. Install provision

Once Drush is installed, install the latest recommended Provision release.

    $ drush dl --destination=/var/aegir/.drush provision-7.x
    $ drush cache-clear drush

#### 4.4. Running hostmaster-install

Once you have downloaded drush and provision, you can install all other aegir components using the hostmaster-install command:

     $ drush hostmaster-install

You will be prompted for the required information if not provided on the commandline. See the inline help for the available options:

     $ drush help hostmaster-install

For example, to install the frontend on Nginx, use:

     $ drush hostmaster-install --http_service_type=nginx

It is imperative that you provide a valid FQDN to the installer. This is used for database GRANTs. Remote web servers depend on the FQDN being resolvable in order to connect back to your Aegir master server if it is used as your database server for managed sites.

Upon completion of the installation, the traditional Drupal 'Welcome' e-mail will be sent to the e-mail address specified by ``--client_email=(your e-mail)`` or if not provided as a command line switch, the address prompted by the installer process. This e-mail address will also be used as the default e-mail address of the first user and client in Aegir, but can be changed later.

There are other commandline switches available, documented in ``drush help hostmaster-install``, as usual.

##### 4.4.1. Arch Linux configuration

    $ drush hostmaster-install --web_group=http

### 5. Install the Hosting Queue Daemon

For Aegir 3.x installs, using the Hosting Queued Daemon (hosting_queued) is highly recommended.

These instructions will install the daemon to run as a regular service in /etc/init.d/. Instructions will vary according to platforms, but the following should work in Debian.

#### 5.1. Install the init script in place

    $ sudo cp <hostmaster_platform_root>/profiles/hostmaster/modules/aegir/hosting/queued/init.d.example /etc/init.d/hosting-queued

#### 5.2. Setup symlinks and runlevels

    $ sudo update-rc.d hosting-queued defaults

#### 5.3. Start the daemon

    $ sudo /etc/init.d/hosting-queued start

### 6. Checkpoint / Finished!

At this point, you have checked out all the code and setup your basic Drupal system (Drupal core, hosting, hostmaster and eldir) that will be the Aegir frontend and the backend system (provision and drush). Your filesystem layout should look something like this:

     /var/aegir/hostmaster-3.x/
     /var/aegir/hostmaster-3.x/profiles/hostmaster/
     /var/aegir/hostmaster-3.x/profiles/hostmaster/modules/contrib/admin_menu/
     /var/aegir/hostmaster-3.x/profiles/hostmaster/modules/contrib/views/
     /var/aegir/hostmaster-3.x/profiles/hostmaster/modules/aegir/hosting/
     /var/aegir/hostmaster-3.x/profiles/hostmaster/modules/aegir/hosting_git/
     /var/aegir/hostmaster-3.x/profiles/hostmaster/themes/eldir/
     /var/aegir/hostmaster-3.x/sites/aegir.example.com/
     /var/aegir/config/server_master/apache.conf
     /var/aegir/config/server_master/apache/conf.d/
     /var/aegir/config/server_master/apache/vhost.d/
     /var/aegir/config/server_master/apache/platform.d/
     /var/aegir/backups/
     /var/aegir/.drush/provision/


Variations on this are acceptable, but you are better to stick with the defaults if you really want to get through this.

The installation will provide you with a one-time login URL to stdout or via an e-mail. Use this link to login to your new Aegir site for the first time.

For troubleshooting this process and resulting install, see the [Installation Trouble-shooting](#installation-trouble-shooting) section.

You may also want to read on with `what you can do with Aegir` now that it is installed.


Installation Trouble-shooting
-----------------------------

### Verify and install through SSH

Since Aegir has multi-server support, it is possible that you have a misconfigured "FQDN" and that aegir then tries to connect to the local server as a remote server. To check if you have a misconfigured server, run the following commands:

     $ resolveip `uname -n`

If the command returns your IP address, you are all set. If it returns an error you will need to edit your `/etc/hosts` file.

First line of this file looks like:

     127.0.0.1  localhost

Simply add all domains you want to this line. e.g:

     127.0.0.1  localhost aegir.example.com example.com

### NameVirtualHost \*:80 has no VirtualHosts

It does not hurt anything, but it can be annoying in your logs. This may disappear as soon as you define your first virtual site using Aegir. If it does not, you most likely have a second NameVirtualHost statement in your configuration someplace other than in the Aegir configurations.

If you are on a Debian system, that is usually a configuration fragment in /etc/apache2/ports.conf or in a fragment symbolically linked in /etc/apache2/sites-enabled and it is safe to comment out any NameVirtualHost statements you find there as this really is a large part of the job you have asked Aegir to do for you.

Once those are commented out, the message should disappear.

### Making sure it works

Your new Aegir server will be installed with a single site named the way you specified in your install script. That's great, but you may have more paths to that same server.

When you try to browse to your server for the first time from a non- localhost browser you may get an addressing issue. If you do, make sure that you actually have the server defined in DNS and that the DNS server was reloaded. If it was reloaded and you use slave servers, make sure that the serial number in the zone file was incremented so that the slaves automatically reload.

If you already have multiple URLs in your DNS which resolve to the same Aegir, you should check them. For instance: if your DNS resolves both aegir.example.com and sitecontrol.example.com to your new Aegir server's IP, you need to make sure that both are accommodated. If they will both get the same physical hostmaster site, one should be set up as an alias to the other. If they are indeed going to be separate sites you will have to create a new site node with the other name as a virtual site.

### Access by the server's physical IP address

Another "gotcha" that you may run into is http access using the actual IP itself. Remember that the IP is not picked up by the site vhost and will not match a ServerName or ServerAlias - so it gets picked up by the default vhost. This is just standard apache stuff, nothing Aegir or Drupal specific here, but quite the annoyance and a potential problem.

You can easily work around it by simply adding the IP as a Site Alias in the site node in either the Aegir site (or another site that you may have defined which you would prefer the IP to address).

You can tell if you have the IP problem by simply pointing your browser to the server by IP as http://999.999.999.999/ and seeing what comes up. If you get an install screen, you have the problem. You certainly do not want that install screen getting executed!

The good news is that fixing it is easy.

Simply log into Aegir, click on Sites, click on the site name that you want the IP to address, click on Edit and scroll down to Domain Aliases. Put the IP into the box, click on the Redirect checkbox so that Aegir instructs Apache to do a rewrite to the real domain name, and finally click Save.

If you do not have the Domain Aliases entry box, you need to turn that feature on. In that case, go to your administration menu at the top of the page, point to Hosting and then click on Features. Click the checkbox for Site Aliasing and then Save Configuration. The Aliases box will now appear when you edit a site.  There is excellent information about the site alias feature in [this handbook page](http://community.aegirproject.org/node/60) with much more detail.

After the Verify job completes you should test your server again. Now when you address the server by it's IP address the URL should automatically change to the site you selected and the install screen never appears.

### CentOS firewall settings

You may need to adjust CentOS's firewall settings to allow HTTP traffic on port 80. If you installed CentOS with a UI, enable "Firewall settings -- WWW (HTTP)".

Alternatively, another solution may be to edit /etc/sysconfig/iptables and add a rule accepting traffic on the relevant interface on port 80.

Afterwards, you can restart the firewall with this command:

    $ sudo service iptables restart

### CentOS cron requires restart?!

Also, in some configurations, it seems necessary to restart crond for the user crontab changes to take effect (very bizarre). For that, use:

    $ sudo service crond restart

See https://www.drupal.org/node/632308 if you have more information about this issue.

### CentOS aegir.conf permission denied

When trying to restart httpd, you may receive:

    Starting httpd: httpd: Syntax error on line 209 of /etc/httpd/conf/httpd.conf: Could not open configuration file
    /etc/httpd/conf.d/aegir.conf: Permission denied

Check your SELinux settings. If enabled, you can run the following command on the aegir.conf file in
/var/aegir/config/server_master directory:

    $ sudo chcon -t httpd_config_t apache.conf

Alternatively, you can disable SELinux completely if you desire. Both options will result in removing the permisson denied error.

You can see [this comment](https://www.drupal.org/node/1286926#comment-5028062) for a little more info.

Try the following if nothing else works:

    $ sudo setenforce permissive

See [this doc](http://wiki.centos.org/HowTos/SELinux) to make the change permanent.

### Solaris cron issues

I had numerous problems setting up a proper cron job, as Solaris' crond seems pretty anal about what it accepts. The only way I could get it to work was to create a wrapper shell script that would be called using the simplest cron tab.

Crontab entry:

    * * * * * /var/aegir/dispatch.sh

Content of dispatch.sh:

    #!/usr/bin/bash
    
    HOME=/var/aegir
    LD_LIBRARY_PATH=/usr/lib:/usr/local/lib:/usr/lib/sparcv9:/opt/mysql/mysql/lib:/usr/sfw/lib:/usr/sfw/lib/gcc:/opt/sfw/lib
    PATH=/usr/bin:/opt/mysql/mysql/bin:/usr/sfw/bin:/opt/sfw/bin:/opt/SUNWspro/bin:/usr/local/bin:/opt/csw/bin
    
    export HOME
    export LD_LIBRARY_PATH
    export PATH
    
    php '/var/aegir/drush/drush.php' --php=/usr/local/bin/php '@hostmaster' hosting-dispatch`


### Drush execution path issues

Solaris (and maybe others) suffers from the dreaded execution issues of drush:

* https://www.drupal.org/node/637574
* https://www.drupal.org/node/586466

Those can be worked around by hardcoding the --php executable on the commandline path. Adding the proper shebang ( `#!/usr/local/bin/php`, for example) header and using a proper PATH that includes the PHP executable also helps.

### APC issues

If you are having trouble running APC with Aegir, try downgrading to APC 3.1.4. This can be achieved by the following:

    sudo pecl uninstall apc
    sudo pecl install apc-3.1.4

