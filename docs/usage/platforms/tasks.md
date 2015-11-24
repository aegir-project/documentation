Platform-specific Aegir tasks
=============================

[TOC]

Platforms, like sites, have tasks that can be performed against them, however these tasks are smaller in number and generally more simple.

Some tasks that are available to platforms out of the box are Lock/Unlock and Delete.


Lock
----

The lock task is simply a conceptual protection placed over a platform so that new sites can't be installed on that platform.

It doesn't actually modify anything on the filesystem or on the platform itself, but merely changes the status of the platform in the Aegir database.  Once a platform is considered 'locked', it no longer appears in the New Site form when adding a new site.

You might choose to Lock a platform if there is something wrong with that platform and you want to prevent new sites from being added to it by one of your clients or users, or you want to 'hide' the platform for some other reason (perhaps awaiting a release date before allowing new sites).


Unlock
------

To unlock a platform that has been locked, simply click the Unlock button and the status of the platform will be switched back to an Enabled state.


Migrate
-------

Platforms have 'migrate' tasks just as sites do. However, in the case of a platform, triggering a migration basically entails automatically adding 'migrate' tasks for all of the sites hosted on the platform. This is a convenient way to keep all of your sites up-to-date, as it's a single task, rather than having to trigger the migrations each site individually.


Delete
------

The Delete task is similar to that of sites. The delete button allows you to completely remove a platform from your filesystem irreversibly.

This can be handy if you engage in a build management methodology that involves making regular new platforms to upgrade your site to. It is easy to amass large numbers of platforms on the filesystem, so the Delete task was born to deal with that problem.

A platform can only be deleted if there are no sites currently provisioned on it. The task will fail before deleting anything if it detects that there are sites in the 'sites' directory of that platform.

You must migrate these sites off to a valid new platform before you can delete such a platform.

Unlike site deletions, there is no backup made of a platform (because it's not bootstrappable by Drush), so be careful with this task: there is no going back!

It is recommended to make regular filesystem backups outside of Aegir altogether, and to use the Lock task to temporarily 'disable' a platform from use if you are unsure whether you want to delete it, but don't want it shown on the Aegir frontend.

### Manually deleting a platform

Similarily to deleting sites, sometimes something goes wrong during a Delete task and the platform doesn't get completely removed. Often this is caused by permissions problems on the platform (i.e something owned by a different user that the aegir user can't remove).

The Delete task sometimes cannot be re-run in this situation. In which case:

* Manually remove the platform files on the server if they exist (i.e `/var/aegir/drupal-7.41`)
* Remove the Drush alias for this platform if it is still present (in `/var/aegir/.drush/`)
* Remove the platform configuration file from `/var/aegir/config/server_master/apache/platform.d` (and the same for the other server\_ directory if the platform was hosted on a remote webserver)
* Go to e.g. /node/123456/delete in your browser on the Aegir frontend and delete the node. This will remove the platform node and all associated task nodes from the system, as well as remove the entry from the hosting_platform and hosting_context table in the database.
