Extending Aegir
===============

[TOC]

Aegir is designed to be easily extendable by developers. As it is made with Drupal and Drush, it is made of the hooks and command you know and love. If you are a user or admin looking to deploy contrib modules, you should look into the contrib modules list and user documentation instead.


What can I extend exactly?
--------------------------

These extensions may come in the form of:

* Adding new tasks to be performed against sites
* Adding new services or implementations of service types (postgres for the DB service, for example)
* Overriding or hooking into existing forms such as the site form, to send extra data to the backend
* Using APIs to inject bits of configuration into configuration files such as settings.php and vhosts.
* Use the powers of Drupal to extend the content types and views.

This area is be devoted to teaching you how to extend and develop for Aegir to encourage contributions to the Aegir project or to help you modify Aegir to suit your unique use case.


Aegir API documentation
-----------------------

The inline documentation is a good start to understand the various hooks and internals that allow you to extend and customize Aegir to your liking. The documentation is rendered on [api.aegirproject.org](http://api.aegirproject.org) daily.

See also the [developer cheat sheet](http://community.aegirproject.org/dev-cheat-sheet/).

Please submit any suggestions or bug reports to the Aegir Project issue queue of your choice, under the "documentation" component.


Example modules
---------------

The [Contributed Extensions](http://docs.aegirproject.org/en/3.x/extend/contrib/) page documents all known Aegir extensions in existence. In addition, Aegir ships with a set of example modules that illustrate how to extend Aegir in various ways.