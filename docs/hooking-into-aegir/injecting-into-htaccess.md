Injecting into platform-wide vhosts (.htaccess)
===============================================

Using a .htaccess with an Allow Override all directive in Apache can be a major performance killer, because it requires Apache to stat each subdirectory of the codebase looking for overrides in .htaccess.

As a result, Aegir disables the reading of the Drupal .htaccess in the runtime environment.

This does not mean that the .htaccess is not needed. Instead, when you run the Verify task against a Platform, the .htaccess is studied by Aegir and its contents are copied into the platform-wide Apache vhost configuration, typically located in /var/aegir/config/server_master/apache/platform.d

Need to make a modification to the .htaccess? Simple: you can simply edit it in-place as you normally would, but you must re-Verify the platform in Aegir afterward, in order for those new or modified settings to be 'loaded in' to the platform vhost file.

The end result is improved performance for your sites, without losing any functionality, as you can still customize the .htaccess to your liking.
																												