Deployment Strategies
=====================

[TOC]

"Deployment strategies" refer to the methods used to deploy platforms in Aegir. Out-of-the-box Aegir only allows for some of these methods.  Others can be enabled after installation.

To see the strategies currently enabled on your system, navigate to *Platforms » Add Platform*.

It should be noted that Aegir will now scan for a `composer.lock` file, and run `composer install`, in order to install a platform's dependencies if any of them are managed by [Composer](https://getcomposer.org/).

Default methods
---------------

### None/Manual/Unmanaged deployment

In the beginning, Aegir did not support any automated build processes for platforms. It assumed that the platform had already been built, and so the platform form only prompted for a local system path. This remains the default, and allows systems outside of Aegir to build platforms without Aegir's knowledge or possible interference.

### Makefile deployment

The first automated build process supported by Aegir was Drush Make. A field is provided to enter the path or URL of a Drush makefile from which to build the platform. While this was the recommended best-practice for Drupal platforms through the 7.x release cycle for a time, Git and/or Composer-based deployments are now more common for Drupal 8+.

### Pure Git deployment

Since multi-site deployments require an immutable platform, an alternative Git deployment mechanism has recently been added. It only performs the initial clone of the platform code-base, and treats it as immutable thereafter.

### Composer deployment from a Git repository

For Drupal 8+. the Drupal community has been moving towards code bases managed entirely with Composer, with no upstream code (e.g. core, contrib modules, contrib patches, etc.) committed to the repository.  We've added this option to support this approach.

Other methods
-------------

### Git-based platform management

With the broad adoption of Git by the Drupal community, the ability to manage platforms using it was added. In order to support advanced development practices, additional features, such as the ability to pull in changes to the platform, and checkout specific commits were added. This is best-suited for hosting single custom sites, since it adds significant risk to multi-site deployments especially in Production environments.  (For those, see **Pure Git deployment** above, a generally safer approach.)

To enable, navigate to *Administration » Hosting » Advanced » Aegir Git*, check the relevant modules for your use case, and then hit the *Save configuration* button.

### Composer deployment from a Packagist repository

In order to host Drupal 8 distributions, a deployment option was added that builds a platform directly from a Composer package on [Packagist](https://packagist.org/). It takes a template project name (and optional version), which it then provides to Composer's `create-project` command.

To enable, navigate to *Administration » Hosting » Optional system features » Aegir Deployment*, check *Composer Deploy via Packagist*, and then hit the *Save configuration* button.
