Sites in Aegir
==============

[TOC]

So, you've provisioned a site using Aegir, and you're poking about looking at the system and what it's done.

A Drupal site managed by Aegir is really mostly the same as any other Drupal site. However, there are a few very minor differences that might surprise or confuse some users.

None of these things adversely affect the running of your site - they are designed to actually make your life even easier.

Multisite
---------

Aegir installs sites on a platform, which means it places the site directory in `/var/aegir/platforms/<your_platform>/sites/<yoursite.com>`. Many sites can all be installed alongside each other inside a platform's 'sites' directory. This is a standard, built-in Drupal feature known as 'multisite', and it is not unique to Aegir.


Settings.php and cloaked credentials
------------------------------------

Within a multisite structure where multiple sites share the same 'Document Root' or codebase on the system, administrative users on sites that have PHP access are capable of manipulating the server into exposing the database credentials of other sites in that codebase with reasonable ease. This is a pretty serious disclosure of sensitive information, although it is only possible if users have PHP execution privileges. Any multisite installation of Drupal (even without Aegir) has the potential for this to be a problem.

But security is very important to Aegir, and that's why special precautions are enabled by default to avoid this issue.

Essentially, when Aegir installs a site and creates the `settings.php` for that site, it 'masks' or 'cloaks' the database credentials, replacing them instead with environment variables fetched from the $\_SERVER array in PHP.

The credentials are instead set in the Web server's configuration file for the site, which is stored outside the 'Document Root' or platform codebase, and is therefore safe from prying eyes by administrative users.

The Web server along with PHP is able to read the environment variables from this configuration and understand the settings.php's mask, allowing your site to work as normal.

If this feature confuses you or is inconvenient, you have a few options available to you:

* The two supported Web servers, Apache and Nginx, cloak the credentials. You can use another Web server.
* Set `$options['provision_db_cloaking'] = FALSE;` in the site's `drushrc.php` and then re-Verify the site.
* Stop worrying and enjoy the extra security Aegir provides you.

Also note that when you run the Backup task against a site, it actually uncloaks the credentials of the settings.php that it saves in a tarball with the rest of the site. The benefit of this is that you are able to take the backup tarball and deploy it on a non-Aegir server, using standard Drupal, and your `settings.php` will be in 'normal' Drupal form for that environment. This also makes it easier to 'import' sites from previous-Aegir installations into new Aegir installations if need be.


'Files' rewrite rule
--------------------

Everyone knows about the inconvenience of having to place an img src in your node that looks like this:

    <img src="/sites/www.example.com/files/mycutecat.jpg">

If you ever deploy this site under a new URL or for whatever reason this path changes, those images or links will become broken on the site. That's because this information ends up stored in the database, and it can be a pain to fix up.

When you Migrate or Clone a site with Aegir, some paths in the database are automatically updated where they are consistent enough to be able to be scripted by Aegir (such as those in the files table).

However, Aegir can't fix up actual node body content, as that would be risky, and as a rule, Aegir doesn't like tampering that deeply into your data. Aegir exists to help you manage your data, but not manipulate it except where absolutely necessary.

Fortunately, what Aegir can offer is adding a '/files' redirect in the Apache vhost configurations. This pattern-matching rule allows you to enter this into your node instead:

    <img src="/files/mycutecat.jpg">

and you can expect that to actually point to the same place. When you clone this site or rename it to a new URL, this path doesn't actually need to change, because the Rewrite rule will be modified to point to the new location, thus avoiding breakages.

It's not perfect, and people are often frustrated with Aegir not 'just fixing' everything like this, despite the fact the above is more than what Drupal does for you out of the box!

Fortunately, it's expected that Drupal 7 will improve this sort of thing altogether and things will hopefully become easier.


The .htaccess
-------------

Using a .htaccess with an Allow Override all directive in Apache can be a major performance killer, because it requires Apache to stat each subdirectory of the codebase looking for overrides in .htaccess.

As a result, Aegir disables the reading of the Drupal .htaccess in the runtime environment.

This does not mean that the .htaccess is not needed. The .htaccess file from the webroot is automatically included by Apache. DO note that other .htaccess files are NOT included.

Need to make a modification to the .htaccess? Simple: you can simply edit it in-place as you normally would, but the remember to reload the Apache configuration.

The end result is improved performance for your sites, without losing any functionality, as you can still customise the `.htaccess` to your liking.

Site specific rules for the site vhost are not directly supported because this can create major security issues. To add your own rules to the site vhost apache config you can use a contrib module like [Hosting Injections](https://www.drupal.org/project/hosting_injections)


Permissions
-----------

Permissions in Aegir generally follow these rules:

* Everything is owned by aegir:aegir, and the 'other' bit is read-only in most cases
* Things that need to be written to (uploads) by the web server, have the 'group' bit of 'www-data' (or whatever the relevant user/group is on your system) with the group bit writable
* Anything that is sensitive information, such as the data in the configuration files, are owned by aegir:aegir or aegir:www-data when the web server needs to read those files, with no read access by the 'other' bit.

The use of the 'aegir' user on the system is vital for normal Aegir functionality, however it can be confusing to users who wish to edit files (say, theme development on a site, or downloading new modules) as a user other than Aegir.

The methodology recommended in these situations is:

* Add a standard user to your server, say, 'john'
* Add that 'john' user to the 'aegir' group: `adduser john aegir`

By default, Aegir sets the 'modules', 'themes' and 'libraries' directories so they are aegir:aegir with the group bit writable. This means that any user on the system who is a member of the 'aegir' group, is then able to add, delete, or modify files within those directories.


The drushrc.php
---------------

You may have noticed a file called 'drushrc.php' on your system, stored alongside the settings.php of a site (and another in the root-level directory of your platform or Drupal core codebase). What's this about?

The drushrc.php is a file that contains important metadata about your site or platform. The significant portion of the file contains metadata on all 'packages' (modules, themes, install profiles and libraries) in that part of the system, including their version numbers and whether they are actually enabled or not.

This file is generated by Aegir whenever a platform is verified, and whenever a site is installed or verified.

Paradoxically, settings from the existing drushrc.php are able to be 'read in' by Aegir when performing those tasks and many others.

The drushrc.php generally should not be edited unless you know what you're doing. As previously discussed above, some supported configurations such as the disabling of the 'cloaking' of database credentials in a site's settings.php, are able to be set in the drushrc.php to provoke the system into modifying how that site is to be treated.


Drush aliases
-------------

Aegir automatically generates a Drush alias for each site that is installed. By
default this alias is named after the site URL. So, for the site at
http://www.example.com, the alias generated by Aegir will be
`@www.example.com`.

Aegir also generates a special alias, `@hostmaster`. This alias represents the
Aegir site itself, and can be useful in triggering tasks from the command line.
A convenient short form of this alias, `@hm` is also provided.
