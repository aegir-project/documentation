Injecting into site vhosts
==========================

Aegir provides some hook functions in its API, one of which allows you to inject extra configuration snippets into your Apache vhost files for sites.

###When would I want to do this?

A good example for this is when you may need to inject a custom Rewrite rule that goes beyond what the Aegir Aliases 'redirection' feature provides.

Or, perhaps you have to inject some htpasswd mod_auth password protection for your site, or perhaps a CustomLog definition. The [http_basic_auth](https://drupal.org/project/hosting_tasks_extra) module uses this. [code](http://cgit.drupalcode.org/hosting_tasks_extra/tree/http_basic_auth?id=HEAD).

Typically you'd just add what you need to the vhost file, but the problem is that Aegir manages these vhosts, and on every Verify task, will rewrite the config from a template, blowing away your changes in the process. Ouch!

Fortunately there is a very easy and elegant solution to this problem to save your configurations persistently across Verify tasks and the like, by means of invoking the Provision hook provision_apache_vhost_config(), or, if you are using Nginx, provision_nginx_vhost_config() (and just replace the below examples with 'nginx' instead of 'apache' where necessary). Below "mig5" is just an example, you can replace this with anything as drush looks for all *.drush.inc files in ~aegir/.drush

###A simple example

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

###A more advanced example

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