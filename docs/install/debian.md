Debian
======

These are the installation instructions that are recommended on Debian. Aegir dependencies (Apache, MySQL, PHP...) are also automatically installed.

If you wish to install Debian packages over an existing manual install, it's possible. See [the Debian upgrade procedures](/upgrading/debian).

Debian packages are uploaded to http://debian.aegirproject.org/ shortly after a release. We eventually want to upload those packages to the official archives, but this will take some adaptation and time to sponsor the packages in.


Requirements for automatic install on Debian
--------------------------------------------

* Basic Linux system administration skills

* Root access to your server

* An up-to-date system and applications (`sudo aptitude update && sudo aptitude safe-upgrade`)

* A configured Fully Qualified Domain Name (FQDN)

  Such as aegir.example.com
  The hostname returned by the commands ``hostname -f`` and ``uname -a`` must resolve to the IP address of your server.

  After setting up your FQDN you must restart your server with a ``reboot`` command so your changes take effect.

  Note that newly created domain name usually take 24 to 48 hours to fully start working. This period, called propagation, is the projected length of time it takes for root name servers and cache records across the entire web to be updated with your website's DNS information.

  See [system-requirements](install/system-requirements)


Adding the project repositories
-------------------------------

Use this command to add the Aegir package "Software Source" repository to your system:

    echo "deb http://debian.aegirproject.org stable main" | sudo tee -a /etc/apt/sources.list.d/aegir-stable.list

To install a customized Debian package, see the [developer instructions for the debian package](/debian). Other distributions are available for courageous people that want to try development versions.

**dev note**: to install the development version of Aegir, you can use the `unstable` or `stable` distribution above.


Adding the archive key to your keyring
--------------------------------------

This repository self-signs packages uploaded to it (and packages uploaded are verified against a whitelist of trusted uploaders) using OpenPGP (GnuPG, to be more precise).

Use these commands to download and add the repository's PGP key, then update the package list on your system:

    wget -q http://debian.aegirproject.org/key.asc -O- | sudo apt-key add -
    sudo apt-get update


DNS configuration
-----------------

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


Manual installation of MySQL (on Ubuntu 12.04 LTS)
--------------------------------------------------

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

Installing Aegir
----------------

To install Aegir version 3, frontend and backend, use the following command:

    $ sudo apt-get install aegir3

This will prompt you for the required information (MySQL password, Postfix configuration...) and go ahead with the install.

During the Postfix configuration, the following options appear: "No configuration, Internet site, Internet with smarthost, Satellite system, Local only". That first text screen only allows to use the tab key to select "OK", and then the enter key to display a second screen where you can select one of the choices. The default is "Internet site", useful in most cases to enable the server to send email messages, for example to the admin.

At the end of the installation, you will receive an email message or, if the user "aegir" has been assigned with a local email account during the installation, the file /var/mail/aegir will contain the message. It will include a one-time login to your new Aegir control panel, that is a URL to copy into your browser so that you can set the password for the "admin" user.

Custom Drupal distributions and make files
------------------------------------------

If you have your own Drupal make file, you can go ahead with the above process, but change the make file to the one you want:

    $ echo "aegir3-hostmaster aegir/makefile string /var/aegir/makefiles/aegir/aegir-custom.make | sudo debconf-set-selections
    $ sudo apt-get install aegir3

This allows you to specify the makefile path for your custom distribution of Aegir. To maintain these customizations, you'll need to ensure you do the same when upgrading.

An example aegir-custom.make file could look like http://cgit.drupalcode.org/provision/tree/aegir-dev.make

After installing Aegir, you can reinstall (WARNING, data is LOST) the front end (hostmaster), with following commands:

    $ sudo rm -rf /var/aegir/hostmaster-\*.\*
    $ sudo su -s /bin/sh aegir -c "drush -y hostmaster-install --aegir_db_pass=$DB_PASSWORD --makefile=$MAKEFILE $DOMAIN"

``su -s /bin/sh aegir -c "some command"`` runs ``some command`` in the ``/bin/sh`` shell as user ``aegir``.  ``sudo`` runs the ``su`` command as root - prompting for your user's password instead ``su`` asking for aegir's password.
 Make sure you fil in the variables.

Troubleshooting the install
---------------------------

To make the install smoother, the install command is run without much debugging information, which can make diagnostics pretty hard. For this, there's a special environment variable you can set that will trigger debugging output. Install aegir with this:


    $ env DPKG_DEBUG=developer sudo apt-get install aegir


You can build your own Debian packages from our repositories using [these instructions](/node/543).
