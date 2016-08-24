Extending Aegir
===============

[TOC]

This section shows how to modify Aegir to suit your unique use case.

Aegir is designed to be easily extendable by developers. As it is made with Drupal and Drush, it is made of the hooks and command you know and love. If you are a user or admin looking to deploy contrib modules, you should perhaps look into the [contrib modules list](/extend/contrib.md) and [user documentation](/usage.md) instead.


What can I extend exactly?
--------------------------

These extensions may come in the form of:

* Adding new tasks to be performed against sites
* Adding new services or implementations of service types (postgres for the DB service, for example)
* Overriding or hooking into existing forms such as the site form, to send extra data to the backend
* Using APIs to inject bits of configuration into configuration files such as settings.php and vhosts.
* Use the powers of Drupal to extend the content types and views.

