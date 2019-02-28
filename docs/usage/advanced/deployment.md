Deployment Strategies
=====================

[TOC]

"Deployment strategies" refer to the method used to deploy a platform in Aegir. Aegir core supports only manual and Drush makefile-based builds by default. However, a number of other options have become available.

By enabling the [Aegir Deploy](https://www.drupal.org/project/hosting_deploy) module, these options are moved into fieldsets controlled by a radio option list, in order to consolidate and clean up the platform node form.

Manual deployment
-----------------

In the beginning, Aegir did not support any automated build processes for platforms. It assumed that the platform had already been built, and so the platform form only prompted for a local system path. This remains the default, and allows systems outside of Aegir to build platforms without Aegir's knowledge or possible interference.

Makefile deployment
-------------------

The first automated build process supported by Aegir was Drush Make. A field is provided to enter the path or URL of a Drush makefile from which to build the platform. This has become the recommended best-practice for Drupal platforms through the 7.x release cycle.

Git-based platform management
-----------------------------

With the broad adoption of Git by the Drupal community, the ability to manage platforms using it was added. In order to support advanced development practices, additional features, such as the ability to pull in changes to the platform, and checkout specific commits were added. This is best-suited for hosting single custom sites, since it adds significant risk to multi-site deployments.

Git deployment
--------------

Since multi-site deployments require an immutable platform, an alternative Git deployment mechanism was recently added. It only performs the initial clone of the platform code-base, and treats it as immutable thereafter.

Composer deployment
-------------------

In order to host Drupal 8 distributions, a new deployment option was added that builds a platform from a Composer package. It takes a template project name (and optional version) which it then provides to Composer's `create-project` command.

### Composer install

While not specific to the `create-project` mechanism, Aegir will now scan for a `composer.lock` file, and run `composer install`, in order to install a platforms dependencies.


