Distribution Updates
====================

[TOC]

The process of automatically building new platforms when needed has also been automated through distributions. There are a variety of update types, including tracking stock distributions published on drupal.org, as well as changes to a Drush makefile or a git repository.

Distribution updates keep an internal "signature" of the current upgrade target. For example, the Drush Makefile plugin stores an MD5 hash of the makefile, whereas the Git plugin stores a commit hash. This allows the updates system to check for the latest source, calculate a new signature, and thus determine whether an update is required.

Each distribution update type plugin includes a queue to regularly check for updates to all such distributions. This queue simply triggers the update check. This, in turn, creates new platforms automatically, whenever an update to a platform is required.

For more responsive update checks, such as kicking off a CI/CD process, webhooks are [under development](https://www.drupal.org/node/2868455).


Update types
------------

### Manual updates

By default, distributions don't assume anything about how their constituent platforms are created. As such, you can continue to use whatever your current mechanism is for keeping your platforms up-to-date. By setting the `upgrade target` fields manually, you still get all the benefits of distributions (`Upgrade` tasks, automatic locking of deprecated platforms, etc.)

### Drush makefiles

Distributions can track the contents of Drush makefiles. By calculating the md5 hash of a makefile and comparing it to the one it recorded last time it updated its platform, it can determine whether a new update is required.

### Git repositories

Git repositories that contain a Drupal code-base can also be used to automatically update distributions. Tracking a Git reference such as a branch (the default) will trigger a new platform build whenever a new commit is added to the tracked branch.

Specific tags or commits can also be referenced though they'll need to be changed manually, in order to update the distribution. The ability to track a Git repository based on tag patterns is currently [under development](https://www.drupal.org/node/2869061).

### Stock distributions

Drupal.org hosts a wide range of distribution projects, along with canonical update data for each. This is used by Drupal core's update check system, as well as Drush Make and other package management commands.

As a result, we can track such "stock" distributions by checking their latest update data against the version of our current upgrade target. As soon as a new release is published, a new platform will be built, set as upgrade target, and all other platforms in the distribution locked.

Behind the scenes, stock updates leverage the Drush makefile update mechanism, by creating a Drush makefile where the stock distribution name, along with version, checksum, and so forth, are injected as a "core" package.

### Composer packages

The Aegir Composer module adds support for building platforms using `composer create-project`. However, Drupal's Composer package publishing system is not yet mature enough to support this. We will support this as soon as it becomes possible. See [this issue](https://www.drupal.org/node/2874000) for the current status of this effort..
