Exporting sites out of Aegir
==========================

[TOC]


Extracting a Drupal site from Aegir
-----------------------------------

You may occasionally want to take a Drupal site hosted on a server managed with Aegir and put it somewhere else. This is pretty easy, but not quite as simple as moving a normal Drupal site not managed by Aegir from one machine to another. In brief, you have to copy the site, delete the `drushrc.php` file, and replace `settings.php` with a copy of the default configuration file. More specifically:

1. Make a backup of your site in Aegir.
2. Download the same copy of drupal core to the new server (and re-apply core patches, if any).
3. Copy `sites/all/` to the new server.
4. Copy the Aegir backup of your site to the new server and unzip in `sites/example.com/`.
5. Create a database on the new server and import the database backup.
6. Copy any custom site aliases in `sites/sites.php` to the new server.
7. Copy any custom configuration in `local.settings.php` to `settings.php`.
8. Edit `settings.php` to add the database authentication details for the new server.

That's it!
