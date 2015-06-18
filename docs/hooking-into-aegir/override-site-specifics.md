Overriding site-specific PHP values
===================================

Sometimes it is useful to override certain PHP values on a per-site basis, but changes to php.ini are generally server-wide. Depending on the value you want to override, a couple of options present themselves.

First, let's consider where PHP values can be changed. The PHP Manual [lists php.ini](http://www.php.net/manual/en/ini.list.php)  directives, and under the "Changeable" column, indicates where a configuration setting may be set. If your value shows either PHP_INI_USER or PHP_INI_ALL, then the easiest way to change this value would be using ini_set() in a local.settings.php file:

    <?php
    @ini_set('memory_limit', '128M');
 
The local.settings.php file should be placed in the root of your drupal site, e.g. /sites/sitename/local.settings.php.   

On the other hand, if the changes mode is either PHP_INI_PERDIR or PHP_INI_SYSTEM, php_ini() won't work. In this case, the solution is to inject the value into the vhost. Since vhosts are managed by Aegir, manually adding an override to /var/aegir/config/server_master/apache/vhost.d/<uri> would get blown away the next time that the site is verified.

As described in [Injecting into site vhosts](#todo), we can inject values into vhosts using a Drush hook. For example, to raise the file upload size limit on http://ergonlogic.com, we add the following code in /var/aegir/.drush/ergonlogiccom.drush.inc:

    <?php
      function ergonlogiccom_provision_apache_vhost_config($uri, $data) {
        if ($uri == "ergonlogic.com") {
          drush_log("Overriding PHP file size values. See .drush/ergonlogiccom.drush.inc");
          return array("php_value upload_max_filesize 100M", "php_value post_max_size 200M");
       }
    }

This results in the insertion of the following lines into /var/aegir/config/server_master/apache/vhost.d/ergonlogic.com:
php_value upload_max_filesize 100M
php_value post_max_size 200M

Also, in the verify task log I get the following informative message:
Overriding PHP file size values. See .drush/ergonlogiccom.drush.inc

###Developer Note

One challenge this technique may present is inspecting the values of the parameters passed into this function. It appears that the Hostmaster site doesn't get bootstrapped, and so common debugging tools (such as devel.module's dd()) aren't available. However drush_log() is, and when called, will push arbitrary messages back into your Aegir site's verify task log.

So sticking the following into the function above can help:
<?php
    drush_log("$uri: " . print_r($uri));
?>