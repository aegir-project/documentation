Common installation problems
============================

1. Verify and install run everything through SSH
------------------------------------------------

Since Aegir has multi-server support, it is possible that you have a misconfigured "FQDN" and that aegir then tries to connect to the local server as a remote server. To check if you have a misconfigured server, run the following commands:

     $ resolveip `uname -n`

If the command returns your IP address, you are all set. If it returns an error you will need to edit your `/etc/hosts` file.

First line of this file looks like:

     127.0.0.1  localhost

Simply add all domains you want to this line. e.g:

     127.0.0.1  localhost aegir.example.com example.com


2. NameVirtualHost \*:80 has no VirtualHosts
--------------------------------------------

It does not hurt anything, but it can be annoying in your logs. This may disappear as soon as you define your first virtual site using Aegir. If it does not, you most likely have a second NameVirtualHost statement in your configuration someplace other than in the Aegir configurations.

If you are on a Debian system, that is usually a configuration fragment in /etc/apache2/ports.conf or in a fragment symbolically linked in /etc/apache2/sites-enabled and it is safe to comment out any NameVirtualHost statements you find there as this really is a large part of the job you have asked Aegir to do for you.

Once those are commented out, the message should disappear.


3. Making sure it works
-----------------------

Your new Aegir server will be installed with a single site named the way you specified in your install script. That's great, but you may have more paths to that same server.

When you try to browse to your server for the first time from a non- localhost browser you may get an addressing issue. If you do, make sure that you actually have the server defined in DNS and that the DNS server was reloaded. If it was reloaded and you use slave servers, make sure that the serial number in the zone file was incremented so that the slaves automatically reload.

If you already have multiple URLs in your DNS which resolve to the same Aegir, you should check them. For instance: if your DNS resolves both aegir.example.com and sitecontrol.example.com to your new Aegir server's IP, you need to make sure that both are accommodated. If they will both get the same physical hostmaster site, one should be set up as an alias to the other. If they are indeed going to be separate sites you will have to create a new site node with the other name as a virtual site.


4. Access by the server's physical IP address
---------------------------------------------

Another "gotcha" that you may run into is http access using the actual IP itself. Remember that the IP is not picked up by the site vhost and will not match a ServerName or ServerAlias - so it gets picked up by the default vhost. This is just standard apache stuff, nothing Aegir or Drupal specific here, but quite the annoyance and a potential problem.

You can easily work around it by simply adding the IP as a Site Alias in the site node in either the Aegir site (or another site that you may have defined which you would prefer the IP to address).

You can tell if you have the IP problem by simply pointing your browser to the server by IP as http://999.999.999.999/ and seeing what comes up. If you get an install screen, you have the problem. You certainly do not want that install screen getting executed!

The good news is that fixing it is easy.

Simply log into Aegir, click on Sites, click on the site name that you want the IP to address, click on Edit and scroll down to Domain Aliases. Put the IP into the box, click on the Redirect checkbox so that Aegir instructs Apache to do a rewrite to the real domain name, and finally click Save.

If you do not have the Domain Aliases entry box, you need to turn that feature on. In that case, go to your administration menu at the top of the page, point to Hosting and then click on Features. Click the checkbox for Site Aliasing and then Save Configuration. The Aliases box will now appear when you edit a site.  There is excellent information about the site alias feature in [this handbook page](http://community.aegirproject.org/node/60) with much more detail.

After the Verify job completes you should test your server again. Now when you address the server by it's IP address the URL should automatically change to the site you selected and the install screen never appears.


5. CentOS firewall settings
---------------------------

You may need to adjust CentOS's firewall settings to allow HTTP traffic on port 80. If you installed CentOS with a UI, enable "Firewall settings -- WWW (HTTP)".

Alternatively, another solution may be to edit /etc/sysconfig/iptables and add a rule accepting traffic on the relevant interface on port 80.

Afterwards, you can restart the firewall with this command:

    $ sudo service iptables restart


6. CentOS cron requires restart?!
---------------------------------

Also, in some configurations, it seems necessary to restart crond for the user crontab changes to take effect (very bizarre). For that, use:

    $ sudo service crond restart

See https://www.drupal.org/node/632308 if you have more information about this issue.


7. CentOS aegir.conf permission denied
--------------------------------------

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


8. Solaris cron issues
----------------------

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


9. Drush execution path issues
------------------------------

Solaris (and maybe others) suffers from the dreaded execution issues of drush:

* https://www.drupal.org/node/637574
* https://www.drupal.org/node/586466

Those can be worked around by hardcoding the --php executable on the commandline path. Adding the proper shebang ( `#!/usr/local/bin/php`, for example) header and using a proper PATH that includes the PHP executable also helps.


10. APC issues
--------------

If you are having trouble running APC with Aegir, try downgrading to APC 3.1.4. This can be achieved by the following:

    sudo pecl uninstall apc
    sudo pecl install apc-3.1.4
