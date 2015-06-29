Remote servers (multiserver)
============================

Aegir supports multiple 'server' entities. These servers have 'services' such as 'Web' or 'Database', and 'service types' which are implementations of that service, such as 'Apache' or 'MySQL'.

Some previous experience in the configuration of Apache and MySQL will help if you want to use remote servers with Aegir. If you haven't had any experience in setting up and maintaining Apache or MySQL before you might want get familiar with the basic concepts first.


### System dependencies

On the remote server, install these packages

    apt-get install rsync apache2 php5 php5-cli php5-mysql postfix mysql-client sudo

### Aegir user

Any number of remote web servers may be configured. The remote server needs an aegir user created on the system.

    adduser --system --group --home /var/aegir aegir
    adduser aegir www-data    #make aegir a user of group www-data

### Web server configuration

You'll also need to prepare the web server in the same way you did for the master Aegir server during installation:

    a2enmod rewrite
    ln -s /var/aegir/config/apache.conf /etc/apache2/conf.d/aegir.conf

Don't restart Apache even when it prompts you. This will be done by the Verify task you'll spawn for the server from the Aegir frontend later.

#### Sudoers

Add the aegir user to sudoers so it can restart Apache

    sudo visudo -f /etc/sudoers.d/aegir):

    aegir ALL=NOPASSWD: /usr/sbin/apache2ctl

### Login shell

The remote aegir user will also need a login shell, which can be modified using the chsh command.

    chsh -s /bin/sh aegir

### MySQL access

Aegir connects directly to the remote server's MySQL to create databases and users. Therefore, you need to log into MySQL in the remote server as root and create a user with the following command:

    GRANT ALL PRIVILEGES ON *.* TO root@aegir_server_ip IDENTIFIED BY 'password' WITH GRANT OPTION;
    FLUSH PRIVILEGES;

Where

* 'root' is the username on the Aegir server that will be connecting. Leave 'root';
* 'aegir_server_ip' is the IP address of your Aegir server. For example '123.123.123.123';
* 'password' is the password that will be used by your Aegir 'root' user to connect to the remote server's MySQL to create databases and users. It is suggested to store that password in a secure location. To increase security, it is recommended to choose a password with 6 to 16 alphanumeric and or punctuation characters. 

If MySQL returns something like "Query OK", that means the command was successful. For example:

    Query OK, 0 rows affected (0.00 sec) 

Note the comment about "bind-address = 127.0.0.1" within section 3.6 of the [manual configuration page](install/manual).

When the server is being verified, Aegir will attempt to create a database and grant privileges to a user for that database. If any of these two fails, [verify that your MySQL configuration is correct](http://www.ghacks.net/2009/12/27/allow-remote-connections-to-your-mysql-server/) and that there is no firewall blocking your MySQL port.

You can confirm that MySQL is configured correctly on the remote server by manually connecting from the command line on your Aegir machine:

    [aegir@aegir_box:~] $ mysql -h <mysql_server_IP> -u root -p

If your MySql database is running on the same server as the Aegir master and is configured with the name "localhost", you may run an issue where the remote web server tries to access MySql via a local socket rather than the hostname. To resolve this, just change the server hostname for the MySql server in Aegir from "localhost" to the hostname of the server.

### SSH keys

SSH public/private keys should be set up so the main Aegir server's aegir user can access remote web aegir users with no passwords required.

Example: on main Aegir server:

    ssh-keygen -t rsa
    (follow prompts)

Do not use a passphrase when you create your key. Simply press Enter to leave the passphrase blank. Otherwise, SSH will insist that you enter the passphrase everytime you try to SSH using the key. We don't want SSH to present any prompts.

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

![Create Server Aegir Admin Screen](/img/create-remote-server.png)

A verify task will be spawned and added to the Task queue ready for dispatching. During a server verification task, various configurations will be set on the Aegir master server and also synced to the remote web server, restarting Apache.

Now when you add a new Platform node in Aegir, you have the option of setting which web server to host it on. If you are not using a makefile but are downloading a platform manually, you must still do this on the main Aegir server. The contents will then be synced across to the web server.

You don't choose a web server when installing a new site. Because a site depends on a platform, its web server is implied by which platform has been chosen. In other words, all sites on a platform are hosted on the same server. You cannot host two sites on the same platform on different servers.