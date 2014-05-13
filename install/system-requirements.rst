.. index:: System Requirements

System Requirements
===================

A system capable of running Drupal
----------------------------------

The Aegir system is entirely Drupal based, and has the same base requirements that Drupal does (with the exception that it won't run on Windows). See more notes on Unix and LAMP/LEMP requirements below.

Your own server
---------------

The low level of access required to be able to configure and run this system is very far beyond what is commonly available to users with shared hosting. A VPS from any popular provider such as Linode, Rackspace, Slicehost, Amazon EC, etc. will do fine. You will need root access to the server and the server needs to be dedicated to Aegir.

A Unix-based operating system
-----------------------------

Aegir must run on some flavour of UNIX, because the majority of functionality in this system occurs in the back-end, through command line scripting. There are also several features (such as symlinks), that are not available to users on Windows. There are no plans currently to add Windows support. See the operating system support page for more information.

Web server
----------

You will need at least one dedicated web server, running Apache. We generally work with Apache 2 but we should be compatible with the 1.x series. Aegir also supports the Nginx web server, but requires at least version 0.7.66 or newer. Since Nginx doesn't provide php-cgi or php-fpm (recommended) modules, you will need to install and run php-fpm server separately. You can find useful examples and tips in the third party Barracuda installer available at the barracuda project page.
N.B.: This third party installer is not supported by the core Aegir developers, but you can find helpful community support at the boa group.

PHP 5.2 and 5.3
---------------

Aegir depends on Drush 4.x, which requires PHP 5.2 or higher. Aegir 2.x depends on a minimun of Drush 5.10 (though Drush 6 is recommended), which requires PHP 5.3 or higher. You also need to have the command-line version of PHP to run Drush properly, and the MySQL extensions for PHP.
Given that PHP 5.2 has been deprecated since July 2010, we suggest using PHP 5.3 if possible. Note that while Drupal 6.x and above support PHP 5.3, some contributed third-party modules may still have problems with this version. Most often these cause warnings that can be safely ignored. Aegir and Drush themselves have no known outstanding PHP 5.3 compatibility issues, although you could have a lot of warnings in Drupal 6 due to ereg deprecation, see this issue for details. If you need to host Drupal 5.x sites, note that Drupal 5.x is not compatible with PHP 5.3 and above, and most likely never will be. See http://drupal.org/node/360605 (amongst other issues) for details. As such, Aegir has dropped support for Drupal 5 in versions 2.0+.

Database server
---------------

You will require a database server, obviously. Aegir currently only supports MySQL and MariaDB. It is preferable to use a dedicated (not shared-hosting) server since Aegir will create database users and will require the use of a MySQL root user.

Mail transfer agent
-------------------

Aegir requires an MTA (Mail Transfer Agent) installed on your webserver in order to be able to install new sites to your new platform. If you don't have an MTA, the site installation will fail with message like "could not send email". Additional messages will show that site has been removed because of this problem. To remedy the situation simply install an MTA like sendmail, postfix, or exim and do the minimal configuration.

Other utilities: sudo, rsync, git and unzip
-------------------------------------------

Aegir installs itself via a Drush Make makefile that downloads via git if you want the bleeding edge code, or via wget if you want the latest official release. If you want the latest development version, and don't have the git program you will need to install it on the server.
The jQueryUI library is used in the Aegir UI, unzip is required to extract it. Sudo is required to allow the aegir user the limited privilege to restart the webserver when required. Rsync is used to sync files to remote servers.

No conflicting Control Panels
-----------------------------

Other popular control panels such as Plesk, cPanel etc, are designed to manage all aspects of Apache configuration and other areas that Aegir also is intended to be used for.
Running Aegir alongside such control panels is not supported and very likely may cause you problems or difficulties installing or running Aegir. Filing bug reports that are caused by interference by another control panel will likely be closed unless the problem can be fixed without causing problems for other Aegir users. Proceed at your own risk!

System requirements of popular Drupal distributions
---------------------------------------------------

Some Drupal distributions, such as OpenAtrium, are specialized products that may contain unique prerequisites for optimal performance. Such examples may include raising the php-cli program's memory_limit to something higher than 64M.
Please note that this is not a requirement of Aegir but of the distribution you are trying to install a site on. Thus the Aegir documentation may not officially 'require' such performance settings, but be aware that Aegir may report errors if the system was under-resourced to complete such a task.
