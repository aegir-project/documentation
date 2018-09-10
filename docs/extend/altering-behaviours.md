Altering Aegir's Behaviours
===========================

[TOC]

Aegir is capable of installing, deploying and moving your sites around, because of this ability, it has to manage the various configuration files that keep your sites running.

These configurations include the standard Drupal settings.php for a site, .htaccess overrides for your HTTP server, as well as HTTP 'VirtualHost' or 'vhost' configuration files that tell your HTTP server where your site is located on the server.

A caveat of this system is that Aegir regularly re-generates these configuration files to apply new changes that have been made to the sites or platforms, as well as doing so as a safety mechanism to ensure these files are 'sane'.

For example, if you make a modification to a site vhost or settings.php and it breaks your site, running a 'Verify' task on the site should restore the file back to how it was.

However, sometimes it is necessary to add custom configuration or overrides to these files, and you can't do that if Aegir is regularly wiping your changes.

Fortunately, Aegir provides a series of hooks and inclusions for overriding or injecting customizations into these files safely and persistently.

This chapter shows you how each of these hooks or inclusions work.

## Customizing Settings.php

Every Drupal website has a file named `sites/DOMAIN.COM/settings.php` that contains important things like database credentials and other settings. Aegir creates this file automatically, so the `settings.php` file is overwritten every time a "verify" task is run on the site. You should never edit it directly.

You can customize your `settings.php` without editing the file itself by creating a settings include file. You can find the files that are included at the bottom of every `sites/DOMAIN.COM/settings.php` file created by aegir:

```php
   # Additional host wide configuration settings. Useful for safely specifying configuration settings.
   if (is_readable('/var/aegir/config/includes/global.inc')) {
     include_once('/var/aegir/config/includes/global.inc');
   }
   # Additional platform wide configuration settings.
   if (is_readable('/var/aegir/platforms/drupal/sites/all/platform.settings.php')) {
     include_once('/var/aegir/platforms/drupal/sites/all/platform.settings.php');
   }
   # Additional site configuration settings.
   if (is_readable('/var/aegir/platforms/drupal/sites/DOMAIN.COM/local.settings.php')) {
     include_once('/var/aegir/platforms/drupal/sites/DOMAIN.COM/local.settings.php');
   }
```

Create a file in the location that fits your needs, and make sure it is php code by including `<?php` at the top.

- **Server-wide:**  All sites hosted on this server will load this code:
  `/var/aegir/config/includes/global.inc`

- **Platform-wide:** All sites using this platform will load this code:
  `/var/aegir/PUBLISH_PATH/sites/all/platform.settings.php`

- **Site-specific:** Only the site indicated by the folder name will load this code:
  `/var/aegir/PUBLISH_PATH/sites/DOMAIN/local.settings.php`

NOTE: These special include files are loaded in this order. This means that **Site-specific** settings override **Platform-wide** settings, which overrides **Server-wide** settings.

Whenever you clone a site or migrate it between platforms, Aegir moves a copy of local.settings.php as well.

Make sure the files can be read by the web server user by changing the file's group to the `web_group` of `server_master`. For example:

```sh
chgrp www-data /var/aegir/platforms/drupal/sites/all/platform.settings.php
```

You can find the correct `web_group` for your system in the `server_master` drush alias at `/var/aegir/.drush/server_master.alias.drushrc.php`.


Overriding PHP.ini settings
---------------------------

Sometimes it is useful to override certain PHP values on a per-site or per-platform basis, but changes to php.ini are generally server-wide. Depending on the value you want to override, a couple of options present themselves.

First, let's consider where PHP values can be changed. The PHP Manual [lists php.ini](http://www.php.net/manual/en/ini.list.php)  directives, and under the "Changeable" column, indicates where a configuration setting may be set. If your value shows either PHP_INI_USER or PHP_INI_ALL, then the easiest way to change this value would be using ini_set() in the `local.settings.php` file:

    <?php
    # Set the memory_limit for this site.
    @ini_set('memory_limit', '128M');
    
    # override Aegir-generated cookie policy for sites - set cookies to expire after a week (604,800 seconds)
    @ini_set('session.cookie_lifetime', 604800);

On the other hand, if the changes mode is either PHP_INI_PERDIR or PHP_INI_SYSTEM, php_ini() won't work. In this case, the solution is to inject the value into the vhost. Since vhosts are managed by Aegir, manually adding an override to /var/aegir/config/server_master/apache/vhost.d/<uri> would get blown away the next time that the site is verified.

As described in the [Injecting into site vhosts](#injecting-into-site-vhosts) section, we can inject values into vhosts using a Drush hook. 

### Developer Note

One challenge this technique may present is inspecting the values of the parameters passed into this function. It appears that the Hostmaster site doesn't get bootstrapped, and so common debugging tools (such as devel.module's dd()) aren't available. However drush_log() is, and when called, will push arbitrary messages back into your Aegir site's verify task log.

So sticking the following into the function above can help:

    <?php
      drush_log("$uri: " . print_r($uri, TRUE));
    ?>

## Programmattically modifying Settings.php

You can also use the API to add module-specific site configurations by adding a drush file with the hook `hook_provision_drupal_config`

    <?php
      * Append PHP code to Drupal's settings.php file.
      *
      * To use templating, return an include statement for the template.
      *
      * @param $uri
      *   URI for the site.
      * @param $data
      *   Associative array of data from provisionConfig_drupal_settings::data.
      *
      * @return
      *   Lines to add to the site's settings.php file.
      *
      * @see provisionConfig_drupal_settings
      */
      function drush_hook_provision_drupal_config($uri, $data, $config) {
        return '$conf[\'reverse_proxy\'] = TRUE;';
      }

For example it could look like this

    <?php
      function drupalwiki_provision_drupal_config($uri, $data, $config) {
        $extra = drush_get_option('site_extra_settings', '');
      // remove window CR
      $extra = str_replace("\r",'',$extra);
        return $extra;
      }

That is used to add the site settings added by the UI in the hosting backend implemented in the [Hosting injections module](https://www.drupal.org/project/hosting_injections)

Injecting into drushrc.php
--------------------------

The drushrc.php file can be changed in two ways:

* changing the templates used for the Drushrc context see the [hook_provision_config_load_templates() hook](http://api.aegirproject.org/api/Provision/provision.api.php/function/hook_provision_config_load_templates/7.x-3.x)
* creating a local.drushrc.php file in the site-specific folder (e.g. sites/example.com/local.drushrc.php

Injecting into site vhosts
--------------------------

Aegir provides some hook functions in its API, one of which allows you to inject extra configuration snippets into your Apache vhost files for sites.

### When would I want to do this?

A good example for this is when you may need to inject a custom Rewrite rule that goes beyond what the Aegir Aliases 'redirection' feature provides.

Or, perhaps you have to inject some htpasswd mod_auth password protection for your site, or perhaps a CustomLog definition. The [http_basic_auth](https://drupal.org/project/hosting_tasks_extra) module uses this. [code](http://cgit.drupalcode.org/hosting_tasks_extra/tree/http_basic_auth?id=HEAD).

Typically you'd just add what you need to the vhost file, but the problem is that Aegir manages these vhosts, and on every Verify task, will rewrite the config from a template, blowing away your changes in the process. Ouch!

Fortunately there is a very easy and elegant solution to this problem to save your configurations persistently across Verify tasks and the like, by means of invoking the Provision hook provision_apache_vhost_config(), or, if you are using Nginx, provision_nginx_vhost_config() (and just replace the below examples with 'nginx' instead of 'apache' where necessary). Below "mig5" is just an example, you can replace this with anything as drush looks for all \*.drush.inc files in ~aegir/.drush

### A simple example

In this example I'll inject a simple 'ErrorLog' apache definition into a vhost to save the site error logs to a file.

Create a file in ~aegir/.drush called mig5.drush.inc. (Choose <any> prefix for the file name <any>.drush.inc).

Add this snippet of PHP to the file:

    <?php
      function mig5_provision_apache_vhost_config($uri, $data) {
         return "ErrorLog /var/aegir/" . $uri . ".error.log";
      }

You can use *any* prefix for the function name *any*_provision_apache_vhost_config

Execute on the server

    drush cache-clear drush

Finally, install a site or verify an existing one

Check your site's vhost config file (in /var/aegir/config/server_master/apache/vhost.d/) and you'll see the line has been injected into the '#Extra configuration' area of the vhost

    <VirtualHost *:80>

      DocumentRoot /var/aegir/hostmaster-HEAD

      ServerName 1.mig5-forge.net
      SetEnv db_type  mysqli
      SetEnv db_name  1mig5forgenet
      SetEnv db_user  1mig5forgenet
      SetEnv db_passwd  X7KzsFhxhp
      SetEnv db_host  tardis
      SetEnv db_port  3306



    # Extra configuration from modules:
      ErrorLog /var/aegir/1.mig5-forge.net.error.log
        # Error handler for Drupal > 4.6.7
        <Directory "/var/aegir/hostmaster-HEAD/sites/1.mig5-forge.net/files">
          SetHandler This_is_a_Drupal_security_line_do_not_remove
        </Directory>

    </VirtualHost>

It's that simple! You can see that via the hook, we pass $uri and the drush data to the function, allowing me to abstract the site url so that each site will get its own error log. You could do extra PHP conditionals to ensure certain data only gets inserted into certain sites of a specific name.

To inject multiple lines instead of one, use an array, i.e

    <?php
      function mig5_provision_apache_vhost_config($uri, $data) {
        return array("ErrorLog /var/aegir/" . $uri . ".error.log", "LogLevel warn");
      }

The key point of this is that the file ~aegir/.drush/mig5.drush.inc file will never be touched by Aegir, so you can rest assured your changes will be respected.

If you want to only inject code into a specific site, wrap your code with an if statement, i.e

    <?php
      function mig5_provision_apache_vhost_config($uri, $data) {
        if ($uri === "<site-name-in-aegir>") {
          return array("ErrorLog /var/aegir/" . $uri . ".error.log", "LogLevel warn");
        }
      }

### A more advanced example

Managing multiple versions of a production site can be a tricky proposition, even in Aegir. This is particularly true when you want to use the same canonical domain name to allow users to access one such site of your choosing at any given moment. For instance, in Aegir I might have a site named www.example.com (with an alias of example.com,) and a couple of alternates with the site names of test1.example.com and test2.example.com. If I suddenly decide that I want my primary domain to access test1.example.com, Aegir forces me into a tedious process that ultimately results in site downtime – I have to delete or migrate the existing www.example.com site to a new unused site name (or clone it to an unused site name, and delete the original) and then I have to migrate test1.example.com to the vacated site name www.example.com. Alternatively, I could add the www.example.com and example.com aliases to test1.example.com but then my users would just be redirected to test1.example.com.

Given the way Aegir manages aliases, it would actually be easier to make this switch if the desired address was never used as a site name to begin with. Instead of having the site name www.example.com alongside of the two other test sites, we might have live.example.com, with in-use aliases of www.example.com and example.com, and a rewrite instruction in the vhost that rewrites live.example.com to www.example.com. If we could easily change the rewrite instructions in that vhost, all we would need to do is move the aliases from one site to the next in the GUI (with the requisite verify tasks executed on each site in question) in order to quickly change which site was being loaded at any given moment by www.example.com. For example, if I wanted to move to test2.example.com, I would remove the www.example.com and example.com aliases from live.example.com and verify, add those aliases to test2.example.com and verify, and change my vhost so that test2 .example.com is rewritten as www.example.com. Since as previously mentioned Aegir overwrites changes to the vhost during the verify process, the solution is to use the provision_apache_vhost_config hook in a drush.inc file to selectively add the rewrite information to the vhost when verifying the site that we want our canonical domain to refer to.

    <?php
      function aliasredirects_provision_apache_vhost_config($uri, $data) {
        // the uri to check here is the name of the site in Aegir
        if ($uri === "test2.example.com") {
            $rval[] = "";
            $rval[] = "# Forces redirect to one uri";
            $rval[] = "RewriteEngine On";
            // if the host is not example.com
            $rval[] = "RewriteCond %{HTTP_HOST} !^example.com$ [NC]";
            // rewrite to example.com with a 301 redirect
            $rval[] = "RewriteRule ^(.*)$ http://example.com$1 [R=301,L]";
            $rval[] = "";
            return $rval;
        }
    }

When we want to switch again, say to test1.example.com, We could update the code to look for the test1.example.com uri.


Injecting into platform-wide vhosts (.htaccess)
-----------------------------------------------

Using a .htaccess with an Allow Override all directive in Apache can be a major performance killer, because it requires Apache to stat each subdirectory of the codebase looking for overrides in .htaccess.

As a result, Aegir disables the reading of the Drupal .htaccess in the runtime environment.

This does not mean that the .htaccess is not needed. Instead, when you run the Verify task against a Platform, the .htaccess is studied by Aegir and its contents are copied into the platform-wide Apache vhost configuration, typically located in /var/aegir/config/server_master/apache/platform.d

Need to make a modification to the .htaccess? Simple: you can simply edit it in-place as you normally would, but you must re-Verify the platform in Aegir afterward, in order for those new or modified settings to be 'loaded in' to the platform vhost file.

The end result is improved performance for your sites, without losing any functionality, as you can still customize the .htaccess to your liking.


Injecting into server-wide vhosts
---------------------------------

Changing maximum filesize upload is common when setting up sites in Drupal. As described in [Overriding site-specific PHP values](#overriding-site-specific-php-values) you can change value for each site created with Aegir by adding a .drush.inc file in /var/aegir/.drush directory. But wouldn´t it be nice to be able to do it server-wide?

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

One further example from staceyb on injecting a rewrite rule. Name the file the same as the function and place it in /var/aegir/.drush

    <?php
      function aliasredirects_provision_apache_vhost_config($uri, $data) {
      // the uri to check here is the name of the site in Aegir
      if ($uri === "example.com") {
        $rval[] = "";
        $rval[] = "RewriteEngine On";
        // check to see if https is not on first
        $rval[] = "RewriteCond %{HTTPS} !=on";
        // rewrite to https with a 301 redirect
        $rval[] = "RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]";
        $rval[] = "";
        return $rval;
      }
    }

Make sure you Verify your site after you create the file. Then scroll through the log and find the message you added in the code.


Running extra code after a task
-------------------------------

Drush provides a number of hooks that Aegir can take advantage of.

There is drush_hook_post_COMMAND(), with an example in hosting_example:

    <?php
      /**
       * Implements drush_hook_post_COMMAND().
       */
      function drush_server_data_post_provision_install() {
        drush_log('server_data received from the frontend (d()->server_data): ' . d()->server_data);
      }

      function drush_server_data_post_provision_verify() {
        drush_log('server_data received from the frontend (d()->server_data): ' . d()->server_data);
      }
    ?>

Running any drush command could then be done like:

    <?php
      provision_backend_invoke('@www.example.com', 'features-revert-all', array(), array('yes' => TRUE));
    ?>


Finding what data is passed to a hook
-------------------------------------

To find out the exact contents of the $data parameter, use a line like below inside your hook.

    <?php
      drush_log('myhook data:' . print_r($data, 1), 'warning');

That would look something like the block below in your task log.

```
   Array (
       [server] => Provision_Context_server Object()

       [application_name] => apache
       [http_pred_path] => /var/aegir/config/server_master/apache/pre.d
       [http_postd_path] => /var/aegir/config/server_master/apache/post.d
       [http_platformd_path] => /var/aegir/config/server_master/apache/platform.d
       [http_vhostd_path] => /var/aegir/config/server_master/apache/vhost.d
       [http_subdird_path] => /var/aegir/config/server_master/apache/subdir.d
       [http_port] => 80
       [redirect_url] => http://example.com
       [db_type] => mysql
       [db_host] => localhost
       [db_port] => 3306
       [db_passwd] => ***
       [db_name] => ***
       [db_user] => ***
       [packages] => Array of package information...
       [installed] => 1
       [config-file] => /var/aegir/platforms/drupal-7.58/sites/example.com/drushrc.php
       [context-path] => /var/aegir/platforms/drupal-7.58/sites/example.com/drushrc.php
       [https_port] => 443
       [extra_config] => # Extra configuration from modules:
   )
```

To find out the return of d() function, use a line like below inside your hook.

    <?php
			drush_log('myhook d():' . print_r(d(), 1), 'warning');

The output would look something like the block below. Note that this is influenced by which modules are enabled.

```
Provision_Context_site Object
(
    [parent_key] => platform
    [name] => @example.com
    [type] => site
    [properties:protected] => Array
        (
            [context_type] => site
            [platform] => @platform_examplepl
            [server] => @server_master
            [db_server] => @server_localhost
            [http_basic_auth_username] =>
            [http_basic_auth_password] =>
            [http_basic_auth_message] =>
            [http_basic_auth_whitelist] =>
            [repo_url] =>
            [deploy_from_git] =>
            [git_ref] =>
            [git_hooks] =>
            [uri] => example.com
            [root] => /var/aegir/platforms/drupal-7.58
            [site_path] => /var/aegir/platforms/drupal-7.58/sites/example.com
            [site_enabled] => 1
            [language] => en
            [client_name] => admin
            [aliases] => Array
                (
                )
            [redirection] =>
            [cron_key] =>
            [drush_aliases] => Array
                (
                    [0] => @example.com
                )

            [profile] => standard
            [install_method] => profile
            [https_enabled] => 0
            [https_client_authentication_enabled] => 0
            [https_key] =>
        )

    [oid_map:protected] => Array
        (
            [platform] => 1
            [server] => 1
            [db_server] => 1
        )

    [service_subs:protected] => Array
        (
            [db] => @server_localhost
        )

)
```
