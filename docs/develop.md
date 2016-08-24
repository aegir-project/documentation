Developing Aegir
================

[TOC]

This devoted to teaching you how to extend and develop for Aegir to encourage contributions to the Aegir project or to help you modify Aegir to suit your unique use case.

Aegir is built on Drupal and Drush, as well as implementing it's own API. As such, a familiarity with the [Drupal API](http://api.drupal.org) and the [Drush API](http://api.drush.org) is pretty much assumed.

This section documents methods and best-practices for hacking on Aegir core or building a [contrib module](extend/contrib.md). You should probably become familiar with the [Extending Aegir](/extend.md) section before proceeding much further.

Aegir API documentation
-----------------------

The inline documentation is a good start to understand the various hooks and internals that allow you to extend and customize Aegir to your liking. The documentation is rendered on [api.aegirproject.org](http://api.aegirproject.org) daily.

See also the [developer cheat sheet](http://community.aegirproject.org/dev-cheat-sheet/).

Please submit any suggestions or bug reports to the Aegir Project issue queue of your choice, under the "documentation" component.


Example modules
---------------

Aegir ships with a set of example modules that illustrate how to extend Aegir in various ways. These are fully functional, and are often used as a template to get the required boilerplate in place.

In addition, the [Contributed Extensions](extend/contrib.md) page documents all known Aegir extensions in existence. Reading through some of these modules' code is a good way to familiarize yourself with how the Aegir API works.
