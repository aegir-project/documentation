.. index:: Manual Installation



Manual Installation
===================

This page describes to process you need to follow if Aegir doesn't
have packages for your distribution. We currently provide :doc:`debian`
packages and others should be coming, if you help! This manual
assumes you are fairly familiar with the UNIX commandline interface,
but should be possible to follow through if you copy and paste
faithfully *all* steps of the procedure.

A note on supported systems:

These instructions provide example commands for a Debian-like
distribution, but should be fairly easy to adapt to other
environments. This document is meant as a canonical reference that
should work on every supported platform. It can also be used for
people porting Aegir to new platforms or installing on alien platform
for which Aegir is not yet packaged.

It currently contains special recommendations for CentOS, RHEL 6, Arch
Linux and Solaris. Users of those platforms are also strongly
encouraged to review the :doc:`common-problems` that occur on
those platforms. Aegir is also known to be installable (and was
developed partly on) Mac OS X, but that process is so obtuse that it
has a `separate page`_ for the first part of the manual (up to Install
Aegir components).

Installing Aegir may seem daunting at first (which is why we provide
automated installs through packages), but once you get around it, it's
fairly simple. It follows those steps:
Table of Contents

[toc]

Note that these instructions setup a complete Aegir system. If you
want to only setup a new remote web/db server, it should be sufficient
to install the system requirements (step 1), configure them (step 2)
and follow the `Remote server how-to`_.



1. Review System Requirements
=============================

See :doc:`system-requirements`


2. Install system requirements
==============================

To install the required components, use your system's package manager. For Debian based systems this would look like:

.. code:: console

  apt-get install apache2 php5 php5-cli php5-gd php5-mysql php-pear postfix sudo rsync git-core unzip


*Note*: replace `apache2` with `nginx php5-fpm` to install nginx on Ubuntu Precise or newer.



3. Configure system requirements
================================


3.1. Create the Aegir user
--------------------------

The provision framework of Aegir requires that the scripts run as a
non-root system account, to ensure that it can correctly set the file
permissions on the hosted files.

Also to ensure that the file permissions of the hosted sites are
always as safe as can be, and especially to make sure that the web
server does not have the ability to modify the code of the site, the
configured system account needs to be a member of the web server
group, in order to be able to correctly set the file permissions.

While you can choose another username, most aegir documentation
assumes the Aegir user is ``aegir``, its home directory is ``/var/aegir``
and the webserver group is ``www-data``.

Shell commands example for Debian bases systems as root:

.. code:: console

  adduser --system --group --home /var/aegir aegir
  adduser aegir www-data    #make aegir a user of group www-data


3.2. Webserver configuration
----------------------------

Aegir supports two popular web servers, Apache and Nginx.


3.2.1. Apache configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Aegir assumes a few Apache modules are available on the server, and
generates its own configuration files. The way we enable this is by
symlinking a single file which contains all the configuration
necessary. In Debian-based systems, you should symlink this file
inside ``/etc/apache2/conf.d`` ior ``/etc/apache2/conf-enabled``  that will be parsed on startup or
alternatively you can place include that file in your
apache.conf/httpd.conf. We prefer the former. In other systems there
are similar ways to accomplish this. Consult your OS's documentation
if unsure.

If you are on a Debian-based system, you will also need to enable the
mod_rewrite module manually.

Run the following shell commands as root. First, configure Apache to
enable RewriteEngine:


::

     a2enmod rewrite


Finally, create a symlink from an apache configuration file to a
folder within the /var/aegir/:


::

     ln -s /var/aegir/config/apache.conf /etc/apache2/conf.d/aegir.conf  


3.2.1.1. Ubuntu 14.04+ specific Apache configuration
````````````````````````````````````````````````````

Ubuntu 14.04 departs from Debian and previous Ubuntu Apache setup in
that it doesn't use ``/etc/apache2/conf.d`` any more and better
separates out ``sites-enabled`` from ``conf-enabled`` configurations. So:


::

     ln -s /var/aegir/config/apache.conf /etc/apache2/conf-available/aegir.conf  
    a2enconf aegir


Do not reload/restart Apache if prompted to after running these
commands, it will fail.


3.2.2. Nginx configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

(If you just succeeded in installing Apache, please skip this
section.)

Aegir assumes standard Nginx configuration is available on the server,
and generates its own configuration files. The way we enable this is
by symlinking a single file which contains all the configuration
necessary. In Debian-based systems, you should symlink this file
inside ``/etc/nginx/conf.d`` that will be parsed on startup.

Please make sure your nginx installation is up and running before
continuing. On Ubuntu 12.04 Server, for instance, you must edit
/etc/nginx/nginx.conf and uncomment the line "types_hash_max_size
2048;" in order for nginx to start successfully.

Shell command as root::


::

  ln -s /var/aegir/config/nginx.conf /etc/nginx/conf.d/aegir.conf


Do not reload/restart Nginx after running these commands, it will
fail.

The installer script creates the configuration file referenced by the
newly created symlink.
Back to top


3.3. PHP configuration
----------------------

Some complex installation profiles or distributions require a PHP
memory limit that is higher than the default. To avoid common errors
when installing sites on some distributions, the PHP command line tool
should be configured to use 192Mb of RAM.

Change the memory_limit directive in /etc/php5/cli/php.ini to read:


::

     memory_limit = 192M      ; Maximum amount of memory a script may consume (192MB)


Most modern Drupal sites require around 96M or even 128M of RAM for
certain operations. This is far more than what is provided by the
default PHP configuration.

Change the memory_limit directive in /etc/php5/apache2/php.ini to
read:


::

  memory_limit = 128M      ; Maximum amount of memory a script may consume (128MB)


If your distributions require more memory than these limits, then use
some common sense and update it as appropriate to suit your individual
needs.



3.4. Sudo configuration
-----------------------

Next, we need to give the aegir user permission to execute the Apache2
command to restart the web server without entering a password.

Create a file at ``/etc/sudoers.d/aegir`` and add the following:


::

    
    Defaults:aegir  !requiretty
    aegir ALL=NOPASSWD: /usr/sbin/apache2ctl


After saving, change the permissions on the file:


::

     chmod 0440 /etc/sudoers.d/aegir


Note - the path to your apache2ctl program may differ from this
example. On some systems it may also be called 'apachectl' instead of
apache2ctl. Adjust to suit your own requirements.



3.4.2. Nginx specific configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For those using Nginx, set the sudoers line as follows


::

     aegir ALL=NOPASSWD: /etc/init.d/nginx



3.5. DNS configuration
----------------------

Aegir requires a properly configured "FQDN" (Fully Qualified Domain
Name) be assigned to the machine. In practice, this means that the
hostname returned by the ``hostname`` and ``uname -n`` shell commands
should resolve to the IP address for this server, and vice versa.

If you only intend to use Aegir on a single server, it is acceptable
for the resolved IP address to be the '127.0.0.1' loopback address.

If you intend to manage multiple servers using Aegir, you will need to
make sure that the IP address is the public IP of this server.

You can add multiple entries to your /etc/hosts file for testing
purposes, for example:


::

     127.0.0.1 aegir.example.com example.com test1.example.com test2.example.com test3.example.com


Then you can use test1.example.com to create your first site.


3.6. Database configuration
---------------------------

Aegir supports MySQL right now. It is best to install the MySQL server
using your Linux distribution's package manager.

Shell commands as root::


::

  apt-get install mysql-server


To make sure that the Aegir backend, and all the possible web servers
can reach your database server, you need to configure mysql to listen
on all the public IP addresses available to it.

Again, as root, edit the MySQL configuration file `/etc/mysql/my.cnf`
configuration line to comment out by placing a # at the beginning of
the line as follow:

Before


::

     bind-address        = 127.0.0.1


After


::

     # bind-address      = 127.0.0.1


Without this line commented out, MySQL will listen only on localhost
for database connection requests.

Now you need to restart mysql, to clear any caches.

Shell command as root:


::

     /etc/init.d/mysql restart


The installer will prompt you for your MySQL root user password. The
root user will be used to make administrative tasks such as creating
new databases, and granting and revoking access to those databases for
sites.

Even though MySQL is now listening on all IP's, it will not allow
invalid users to connect to the databases, without the correct user
accounts configured.

If you are concerned about MySQL being accessible in this way, you can
also configure your firewall to only allow incoming connections from
certain addresses. This is outside the scope of this document however.

Note that Aegir will ask you for your MySQL root password. If you do
not want to use your regular root password for Aegir, you will need to
create another root account for Aegir using a MySQL command like:


::

     GRANT ALL PRIVILEGES ON *.* TO 'aegir_root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;


Note: If you are running your Aegir databases on a remote DB server,
you will want to create this aegir_root user. The install will often
fail if you're trying to use the root user on a remote database. See
`this issue`_ for details.
Back to top


3.6.1. Ubuntu, RHEL, Arch linux specific configurations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

NOTE : If you are running either Ubuntu 12.04 LTS, RHEL 6 or Arch
Linux, you should still install MySQL in the same way as above.
However, once done, you must now remove the anonymous, passwordless
login that those platforms creates by default. To do this, run:


::

     `sudo mysql_secure_installation
    `


Otherwise, Aegir will fail to install and work at all. See `this FAQ
entry`_.
Back to top


4. Install Aegir components
===========================

Next step is to install the Aegir software components themselves, that
is: drush, provision and hostmaster.


4.1. Install drush
------------------

Before installing Aegir proper, you first need to install Drush.

See then Drush documentation for this: http://docs.drush.org/en/master/install/#composer-one-drush-for-all-projects



4.2. Stop! Now become the Aegir user!
-------------------------------------

The remaining of this manual assumes you are running as the Aegir
user. Things will go very wrong if you do not change your shell
credentials to become that user. You can do this by running the
following command as root:


::
    
    su -s /bin/bash - aegir


If this fails because `/bin/bash` doesn't exist, try using `/bin/sh`.


4.3. Install provision
----------------------

Once Drush is installed you should be able to install the latest
recommended Provision release using the following drush command:


::

    
    drush dl --destination=/var/aegir/.drush provision-7.x-3.0
    drush cache-clear drush



4.4. Running hostmaster-install
-------------------------------

Once you have downloaded drush and provision, you can install all other aegir components
using the hostmaster-install command:


::

     drush hostmaster-install


You will be prompted for the required information if not provided on
the commandline. See the inline help for the available options:


::

     drush help hostmaster-install


For example, to install the frontend on Nginx, use:


::

     drush hostmaster-install --http_service_type=nginx



It is imperative that you provide a valid FQDN to the installer. This
is used for database GRANTs. Remote web servers depend on the FQDN
being resolvable in order to connect back to your Aegir master server
if it is used as your database server for managed sites.

Upon completion of the installation, the traditional Drupal 'Welcome'
e-mail will be sent to the e-mail address specified by
``--client_email=(your e-mail)`` or if not provided as a command line
switch, the address prompted by the installer process. This e-mail
address will also be used as the default e-mail address of the first
user and client in Aegir, but can be changed later.

There are other commandline switches available, documented in ``drush
help hostmaster-install``, as usual.


4.4.1. Arch Linux specific configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


::

    
    drush hostmaster-install --web_group=http



5. Install the Hosting Queue Daemon
===================================

For Aegir 3.x installs, using the Hosting Queued Daemon
(hosting_queued) is highly recommended.

These instructions will install the daemon to run as a regular service
in /etc/init.d/. Instructions will vary according to platforms, but
the following should work in Debian, running as root.


#. Install the init script in place

.. code:: console

  cp <hostmaster_platform_root>/profiles/hostmaster/modules/hosting/queued/init.d.example /etc/init.d/hosting-queued


#. Setup symlinks and runlevels

.. code:: console

  update-rc.d hosting-queued defaults


#. Start the daemon

.. code:: console

  /etc/init.d/hosting-queued



6. Checkpoint / Finished!
=========================

At this point, you have checked out all the code and setup your basic
Drupal system (Drupal core, hosting, hostmaster and eldir) that will
be the Aegir frontend and the backend system (provision and drush).
Your filesystem layout should look something like this:


::

    
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


Variations on this are acceptable, but you are better to stick with the
defaults if you really want to get through this.

The installation will provide you with a one-time login URL to stdout
or via an e-mail. Use this link to login to your new Aegir site for
the first time.

For troubleshooting this process and resulting install, see the
`common installation problems page`_.

You may also want to read on with `what you can do with Aegir`_ now
that it is installed.
