Injecting into server-wide vhosts
=================================

Changing maximum filesize upload is common when setting up sites in Drupal. As described in [Overriding site-specific PHP values](site-specific-overrides) you can change value for each site created with Aegir by adding a .drush.inc file in /var/aegir/.drush directory. But wouldn´t it be nice to be able to do it server-wide?

For instance, you could create a file called global_settings.drush.inc, place it in /var/aegir/.drush and put in the following code:

    <?php
      function globalsettings_provision_apache_vhost_config($uri, $data) {
      drush_log("Overriding PHP file size values. See .drush/global_settings.drush.inc");
      return array("php_value upload_max_filesize 100M", "php_value post_max_size 200M");
    }

Here below is the same code from ergonlogic demonstrating how to create a domainname.drush.inc file using the domain name as a condition before the code injection. This then would only affect the specific site and not apply server wide.

    <?php
    function ergonlogiccom_provision_apache_vhost_config($uri, $data) {
    if ($uri == "ergonlogic.com") {
      drush_log("Overriding PHP file size values. See .drush/ergonlogiccom.drush.inc");
      return array("php_value upload_max_filesize 100M", "php_value post_max_size 200M");
     }
    }

Make sure you Verify your site after you create the file. Then scroll through the log and find the message you added in the code.