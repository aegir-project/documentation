Hooking into Aegir (site overrides)
===================================

Aegir is capable of installing, deploying and moving your sites around, because of this ability, it has to manage the various configuration files that keep your sites running.

These configurations include the standard Drupal settings.php for a site, .htaccess overrides for your HTTP server, as well as HTTP 'VirtualHost' or 'vhost' configuration files that tell your HTTP server where your site is located on the server.

A caveat of this system is that Aegir regularly re-generates these configuration files to apply new changes that have been made to the sites or platforms, as well as doing so as a safety mechanism to ensure these files are 'sane'.

For example, if you make a modification to a site vhost or settings.php and it breaks your site, running a 'Verify' task on the site should restore the file back to how it was.

However, sometimes it is necessary to add custom configuration or overrides to these files, and you can't do that if Aegir is regularly wiping your changes.

Fortunately, Aegir provides a series of hooks and inclusions for overriding or injecting customizations into these files safely and persistently.

This chapter shows you how each of these hooks or inclusions work.