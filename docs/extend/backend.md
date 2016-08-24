Provision - the Aegir backend
=============================

[TOC]

The Aegir backend is a Drush extension called "Provision". It does all the heavy lifting in Aegir, such as writing vhost files and creating new databases. You can use additional Drush extensions to extend Aegir or [modify its behaviour](/extend/altering-behaviours.md).


Adding a Drush extension
------------------------

There are a couple ways that you can add a Drush extension. First, and most commonly, you can add it under `/var/aegir/.drush/` and clear the Drush cache. Drush will search a number of paths for extensions, but that's the the conventional place to put them on an Aegir install.

Since Aegir 3, you can also bundle Drush extensions with front-end modules that expose a "Hosting Feature." This allows for both the frontend module, along with the included backend extension, to be deployed via a makefile, as part of the Hostmaster platform. Upon enabling such a Feature, Aegir will re-write it's drushrc file to dynamically include the backend directory for each Hosting Feature.

