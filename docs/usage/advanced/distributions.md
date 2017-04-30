Distributions
=============

[TOC]

The Aegir Distributions module simplifies and further automates platform maintenance and site updates.

It extends the Aegir Hosting System to add a concept of "distributions". According to [drupal.org](https://www.drupal.org/project/project_distribution):

> Distributions provide site features and functions for a specific type of site as a single download containing Drupal core, contributed modules, themes, and pre-defined configuration. They make it possible to quickly set up a complex, use-specific site in fewer steps than if installing and configuring elements individually.

In Aegir terminology, a distribution is a "type" or "flavour" of platform (i. e., code-base). It could just as easily be an off-the-shelf package like OpenAtrium or Thunder, as a custom cobe-base specific to a client's use-case.


Background
----------

At its most basic, distributions are intended to simplify updating sites to the latest platform that is appropriate for them. It reduces the amount of contextual knowledge required to keep sites up-to-date.

Let's take a minute to explore the problems that distributions help to solve. For the sake of simplicity, let's assume that we have just a basic single-server Aegir installation, on which we're hosting a couple Drupal 7 and a couple Drupal 8 sites.

When a Drupal Security Advisory is released for Drupal 8, we need to build an updated D8 platform, then migrate our sites from the old D8 platform to the new one.

We first take a look at the existing platform node, which contains crucial information, such as the Drush makefile or Git repository from which the platform was built. Based on that, we can take the appropriate actions to build an up-to-date platform (i.e., update a makefile, merge an upstream git repository, etc.)

We then need to create a new platform node. We'll want to be careful about how we name it, so as to provide clear indications for later steps. One common practice is to append a unique identifier to the platform name. This could be:

* a version number (e.g. `Drupal 7.54`);
* a date-stamp (e.g. `Drupal 7 2017-03-14`); or a
* build iteration (e.g. `Drupal 7 build123`).

Once an up-to-date platform is ready, we then need to go back to the old platform, and trigger a 'migrate' task. Alternatively, we could visit each site individually and trigger 'migrate' tasks. Either way, while `migrate` is an effective description of the process, it is not at all intuitive that this is the process used to update a site. This is a common source of confusion for users new to Aegir.

Once we click the 'migrate' button, we're presented with a list of platforms to choose from. Here is where our platform naming from earlier becomes important. Hopefully the appropriate choice is reasonably obvious. However, in addition to the intended upgrade target, we'll also be offered, at least, the Drupal 8 platform, in this scenario. If you maintain multiple types of platforms, this choice can quickly become confusing, and lead to errors from which it can be difficult to recover.

If we wanted to update sites individually, then we'd need to repeat this possibly error-prone process for each of them. We're also offered the chance to change the site's URL, effectively renaming it. While certainly helpful functionality to have available, it is, again, just a distraction to our goal of updating the sites.

Let's next take a look at how distributions help with this scenario.

Basic Usage
-----------

Once enabled, this module provides a 'distribution' content-type, as well as adds a "Distribution" field to platform nodes. To begin, we'll create a distribution node (/node/add/distribution). Let's call it "Drupal7" for this example.

Then, we go back to each of our existing Drupal 7 platforms, edit them, and select "Drupal7" in the distribution field. This will also add a link to our distribution when we view these platforms. Visiting the distribution again, we'll now see a list of the platforms we just added.

If we then edit the "Drupal7" distribution node, we'll see a new `upgrade target` field. Here we select the newer platform that we just created earlier. We still have to be careful here, as decribed above. However, we only need to make this selection once, whenever we add a more up-to-date platform. With the addition of the "distribution updates" functionality described in the "Advanced Usage" section, the upgrade target is even updated for us, further simplifying this process.

Having designated an upgrade target, all other platforms using this distribution are automatically locked, making it impossible for new sites to be built on out-of-date platforms.

In addition, any platforms using this distribution will now "know" where it's sites need to be migrated in order to be updated. As a result, a new `Upgrade` task is now available on the out-of-date platforms, and all sites hosted on those platforms.

Running an `Upgrade` task provides a summary of the updates that will take place, and provides "Compare" links that will give details in a side-by-side table. If you were to click the "Confirm" button, a migrate task would be scheduled behind the scenes, that automatically specifies the "upgrade target" platform.

The `Upgrade` task is only available when appropriate. So, for example, sites on the "upgrade target" platform, are already up-to-date (by definition), and so the `Upgrade` task is disabled.


Advanced Usage
--------------

Automating the process of building new platforms is currently under development. This feature will include a queue to regularly check for updates to a platform, as well as webhooks, that will allow for new platforms to be created automatically, whenever a updates to a platform are required.






