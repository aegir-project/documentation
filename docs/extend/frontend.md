Hosting - the Aegir frontend
=============================

[TOC]

The Aegir frontend is a Drupal site in itself. You can use any Drupal module to extend it.



Adding a module
-------------

Add a new module either to the site specific moduled directory e.g. `/var/aegir/hostmaster-7.x-3.x/sites/aegir.example.com/modules` or use [your own makefile](/install/#711-custom-make-files) to create the hostmaster platform.


UI modules
----------

To change one of the content types of views used in the Aegir frontend make sure you have the views_ui and field_ui core modules enabled.
You can enable the like any other Drupal module as they are included with code already.
