Contributed extensions
======================

[TOC]

Note that the support and development status varies for these. Please evaluate each extension for yourself (or [ask the advice of an expert]()) before installing them on a production Aegir system.


"Golden" Contrib
----------------

### Included in Aegir package

These ship with Aegir (the [Golden Contrib](http://cgit.drupalcode.org/hostmaster/tree/drupal-org.make) suite of modules) so their features simply require enabling if desired.  See [the guidelines](http://docs.aegirproject.org/en/3.x/develop/repositories#golden-contrib-module-guidelines) for inclusion.  For the others, manual installation is required.

#### [Hosting CiviCRM](https://www.drupal.org/project/hosting_civicrm)

Module to configure settings and cron jobs specific to CiviCRM.

#### [Hosting Git](https://www.drupal.org/project/hosting_git)

This is a simple module for the Aegir project that adds a 'Git pull', 'Git checkout' and 'Git clone' task for sites.  It's the successor of Hosting Site Git & Hosting Platform Git.

#### [Hosting Remote Import](https://www.drupal.org/project/hosting_remote_import)

Provides a UI for fetching sites from remote Aegir servers.

#### [Hosting Site Backup Manager](https://www.drupal.org/project/hosting_site_backup_manager)

Extends the backup functionality of Aegir. It adds a tab to the site content type. The tab shows the backups and enables per backup actions (Restore, Delete and Get).

#### [Hosting Tasks Extra](https://www.drupal.org/project/hosting_tasks_extra)

This module extends Aegir hostmaster (and drush/provision) with some additional tasks:

* Revert features
* Flush all caches
* Rebuild registry
* Run cron
* Sync data (between sites)
* Run DB schema updates

It also includes site functionality for:

* [HTTP basic authentication](https://github.com/computerminds/aegir_http_basic)
* Generating and showing Drush aliases


Front-end Extensions
--------------------

While these modules are not included in the base Aegir distribution, they can be added easily, since the are (mostly) just Drupal modules. As such, they can be installed in the [usual way](https://www.drupal.org/docs/7/extending-drupal-7/installing-contributed-modules).

However, we recommend instead maintaining a custom Aegir makefile that can include any of these modules (or [themes](#themes)). Such custom Aegir makefiles are supported by:

* Debian package [installation](/install/#711-custom-make-files) and [upgrades](/install/upgrade/#custom-distributions).
* manual [installation](/install/#44-running-hostmaster-install) and [upgrades](/install/upgrade/#upgrading-the-frontend), via the `--makefile` option.
* most of the suggested [configuration management tools](#configuration-management).


#### [Aegir Cloud](https://www.drupal.org/project/aegir_cloud)

Aegir Cloud allows Aegir to create servers directly in cloud hosting providers like IBM Softlayer and Amazon Web Services.

#### [Aegir Config Management](https://www.drupal.org/project/aegir_config)

This module provides config-export and config-import commands for Aegir.

#### [Aegir Feeds](https://www.drupal.org/project/aegir_feeds)

Provides [Feeds](https://www.drupal.org/project/feeds) integration with Aegir.

#### [Aegir HTTPS](https://gitlab.com/aegir/hosting_https)

This module enables HTTPS support for sites using certificate management services such as [Let's Encrypt](https://letsencrypt.org/), whose support is included.

It provides a cleaner, more sustainable and more extensible implementation that what's currently offered in Aegir SSL within Aegir core, and doesn't require workarounds such as [Hosting Let's Encrypt](https://github.com/omega8cc/hosting_le).

#### [Aegir Kubernetes](https://gitlab.com/aegir/hosting_kubernetes)

This module adds [Kubernetes](http://kubernetes.io) support by allowing for hosting of any [containerized](https://en.wikipedia.org/wiki/Operating-system-level_virtualization) application (via [Docker](https://en.wikipedia.org/wiki/Docker_(software)) for now) with resource definition files.

#### [Aegir Network](https://www.drupal.org/project/hosting_network)

Allow inter-communication between Aegir servers (to address the "smart nodes" use case). The point is to centralize information to facilitate management of multiple servers.

#### [Aegir Reporting](https://www.drupal.org/project/hosting_report)

This module provides a reporting framework. It builds atop the [Aegir Monitoring API](https://www.drupal.org/project/hosting_monitor) to report on the health of hosted sites. It ships with some basic probes, but is intended to be extended, as with [Aegir Site Audit](https://www.drupal.org/project/hosting_site_audit).

#### [Aegir Rules](https://www.drupal.org/project/aegir_rules)

Provides integration with [the Rules module](https://www.drupal.org/project/rules) to enable actions on various triggers.  Related to [Hosting Notifications](https://www.drupal.org/project/hosting_notifications).

#### [Aegir Services](https://www.drupal.org/project/hosting_services)

Aims to be a one-stop shop for all Web services functionality offered within Aegir. It allows for remote site management via the [Services](https://www.drupal.org/project/services) framework.

The Aegir SaaS sub-module sets up a fully functional endpoint (via the base module's API) allowing for remote administration of sites, notably cloning existing sites, for software-as-a-service (SaaS) / site-factory Aegir set-ups. It fully configures a service endpoint providing common parameters for cloning as configured in the module's settings. Using the API's task resource, sites can also be disabled, enabled, deleted, and have any other task performed on them supported by your Aegir installation. See [the module's README](http://cgit.drupalcode.org/hosting_services/tree/submodules/hosting_saas/README.md) for more information.

#### [Aegir Site Probes](https://www.drupal.org/project/hosting_probes)

This module fetches information from sites for use in other hosting modules.

#### [Hosting Dev](https://www.drupal.org/project/hosting_dev)

This is a suite of tools to enable smoother development using Aegir. It is currently simply a port of the hosting_reinstall module to Aegir 3.

#### [Hosting Drush Backup](https://www.drupal.org/project/hosting_drush_backup)

Provides a provision command and a task in the hostmaster interface to create backups using drush archive-dump instead of the normal provision-backup code.

#### [Hosting DNS](https://www.drupal.org/project/hosting_dns)

DNS server integration for Aegir, previously included in core.

#### [Hosting Drulenium](https://www.drupal.org/project/hosting_drulenium)

This module adds [Drulenium](https://www.drupal.org/project/drulenium) tasks to be performed on an Aegir managed site.

#### [Hosting Injections](https://www.drupal.org/project/hosting_injections)

Adds fields for injecting custom settings into settings.php and the Apache vhost.

#### [Hosting Let's Encrypt](https://github.com/omega8cc/hosting_le)

*Deprecated. Use [Aegir HTTPS](https://gitlab.com/aegir/hosting_https) instead.*

This module replaces self-generated Aegir certificates with Let's Encrypt ones.


#### [Hosting Logs](https://www.drupal.org/project/hosting_logs)

This is a simple module for the Aegir project that adds a 'Logs' tab to sites and platforms. Showing Apache error, Git commit and watchdog logs.

#### [Hosting Migrate Module](https://www.drupal.org/project/hosting_migrate_module)

This module allows to launch migrate import through the Aegir interface.


#### [Hosting Notifications](https://www.drupal.org/project/hosting_notifications)

Integrates with [the Notifications framework](https://www.drupal.org/project/notifications). This allows you to receive notifications about task execution in various formats, Email, Twitter, iPhone etc.  Related to [Aegir Rules](https://www.drupal.org/project/aegir_rules).

#### [Hosting Piwik](https://www.drupal.org/project/hosting_piwik)

Integrates with an instance of the [Piwik analytic software package](https://en.wikipedia.org/wiki/Piwik) and the [Drupal Piwik module](https://www.drupal.org/project/piwik).

#### [Hosting Site Make](https://github.com/mglaman/hosting_site_make)

Allows a site to have its modules built from a makefile in the sites directory.

#### [Hosting Task Graphs](https://www.drupal.org/sandbox/kienan/2735595)

Adds graphs to visualize task duration evolution over time for Aegir hosting tasks.

#### [Hosting Variables](https://www.drupal.org/project/hosting_variables)

Allows you to set arbitrary custom Drupal variables for each site, such as site name and slogan.  These variables will be put in settings.php so can't be overriden (or changed) through the site interface.

#### [Hosting Wordpress](https://www.drupal.org/project/hosting_wordpress)

Module to manage WordPress sites. It aims to support the main Aegir functionality, such as installation, upgrade, migration and backups.


Extensions to Provision (backend)
---------------------------------

Starting from Aegir 7.x-3.x the Drush component can be included in a 'drush' directory in the same git repository as the hosting module.

Therefore this list will be shorter then for previous versions.

### [Provision STS](https://github.com/mlutfy/provision_sts)

Adds the Strict Transport Security header to hosts that require SSL.


Extensions to client sites
--------------------------

These modules are not for hostmaster, but for the sites hosted under Aegir.

### [Aegir Pathologic Files](https://github.com/Lab43/aegir-pathologic-files)

A tiny Drupal Module to simplify file paths in content. This helps prevent broken images when the site directory name changes. Requires an apache rewrite rule to point /files to /sites/example.com/files, which Aegir provides by default.

### [Site Quota Enforcer](https://www.drupal.org/project/quenforcer)

Enforces quotas set for the entire Drupal site. It works by preventing new entities from being creating on form submission if associated resources are at or past their limits.

Currently supported limits are the number of users and the amount of storage (the combined files and database size).


Themes
------

*   [Saeven](https://www.drupal.org/project/saeven)
*   [Eldir](https://www.drupal.org/project/eldir). (The default theme)
*   [Bara](https://github.com/MartijnBraam/aegir-bara)

Configuration management
------------------------
Those projects allow you to manage Aegir instance(s) through a configaration management tool.

*   [Aegir Puppet module](https://www.drupal.org/project/puppet-aegir)
*   [Drush Puppet module](https://www.drupal.org/project/puppet-drush)
*   [Aegir3 Chef Cookbook](https://supermarket.chef.io/cookbooks/aegir3)
*   [Ansible role](https://github.com/GetValkyrie/ansible-role-aegir/)
*   [Aegir Ansible](https://www.drupal.org/project/aegir_ansible)


Development
-----------

### [Valkyrie](https://www.drupal.org/project/valkyrie)

Valkyrie is an opinionated development stack that makes features/git based Drupal development easy.

Features include:
*   Everything is wrapped up neatly in a VM. This keeps your computer tidy and Valkyrie consistent across various machines.
*   Folders in the VM are mounted from your computer via NFS to make developing with your favorite editor easy (we like Vim).
*   Automatic domain resolution using vagrant-dns on Macs or Avahi on Linux systems (we haven't tested this on Windows, sorry). Each site you create on Valkyrie will get an automatically resolving domain name which keeps you from needing to hack your /etc/hosts file.
*   Drush extensions to make all kinds of common development tasks easy.
*   Automatic Drush aliases for running commands against sites inside the VM.
Development & Support

While provide release on drupal.org, mostly to allow for installation via: drush dl valkyrie, development happens on Github at: [GetValkyrie](https://github.com/GetValkyrie/valkyrie)


### [Aegir Development Environment](https://github.com/aegir-project/development)

A couple of scripts for easier Aegir development using Docker.

### [Aegir up](https://www.drupal.org/project/aegir-up) (deprecated)

*This tool is deprecated, having been replaced by [Valkyrie](#valkyrie), above.*

Aegir-up is a Drush extension that deploys a local instance of the Aegir Hosting System atop Vagrant and Virtualbox, for development and testing purposes.


Others
------

### [Skynet](https://github.com/PraxisLabs/skynet)

Skynet is an experimental replacement for the Aegir Hosting System's Queue Daemon. It is written in Python using [Cement](http://builtoncement.com).


### [DevShop](https://www.drupal.org/project/devshop)

DevShop is a Drupal Development Environment Manager built on Aegir.

DevShop creates Aegir platforms and sites automatically from Git URLs. It tracks multiple Projects and allows multiple environments to be created for each Project, such as dev, test, and live. It provides tools to Pull Code, Sync Data, Commit Features, and Run Tests on these environments, and provides a dashboard with useful links and information for developers.

### [Recurly Aegir](https://www.drupal.org/project/recurly_aegir)

Automatically manages [Aegir](https://www.drupal.org/project/hostmaster) sites from another Drupal instance with [Recurly](https://www.drupal.org/project/recurly) subscriptions / recurring billing / e-commerce.  Communicates with an Aegir instance via [Aegir SaaS](#aegir-services).

This Drupal 8+ module allows clients to pay for their hosted Drupal sites. When a new subscription notification from Recurly is received, a new site will be provisioned. If a subscription is [cancelled](https://dev.recurly.com/page/webhooks#section-canceled-subscription) or [expired](https://dev.recurly.com/page/webhooks#section-expired-subscription), the site will be disabled. If it gets [reactivated](https://dev.recurly.com/page/webhooks#section-reactivated-account) or [re-added](https://dev.recurly.com/page/webhooks#section-new-subscription), the site will be re-enabled. Sites can be deleted after subscription expiration if not renewed.

Your extension here?
--------------------

_Developers:_ Please add your contributed extensions here. Pull requests welcome on [GitHub](https://github.com/aegir-project/documentation/).
