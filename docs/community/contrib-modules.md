Please note that the support and development status varies for these contributed modules.

## 1. Extensions to Hostmaster (frontend)

### [Hosting CiviCRM](https://www.drupal.org/project/hosting_civicrm)

Module to configure settings and crons specific to CiviCRM.

### [Hosting Drulenium](https://www.drupal.org/project/hosting_drulenium)

This module adds [Drulenium](https://www.drupal.org/project/drulenium) tasks to be preformed on an Aegir managed site.

### [Hosting Git](https://www.drupal.org/project/hosting_git)

This is a simple module for the Aegir project that adds a 'Git pull', 'Git checkout' and 'Git clone' task for sites.
The successor of Hosting site Git & Hosting platform Git

### [Hosting Logs](https://www.drupal.org/project/hosting_logs)

This is a simple module for the Aegir project that adds a 'Logs' tab to sites and platforms. Showing Apache error, Git commit and watchdog logs.

### [Hosting Piwik](https://www.drupal.org/project/hosting_piwik)

Integrates with an instance of the Piwik analytic software package and the Drupal piwik module.
D7 port in https://www.drupal.org/node/2195591

### [Hosting Remote Import](https://www.drupal.org/project/hosting_remote_import)

Provides a UI for fetching sites from remote Aegir servers.

### [Hosting Site Backup Manager](https://www.drupal.org/project/hosting_site_backup_manager)

Extends the backup functionality of Aegir. It adds a tab to the site content type. The tab shows the backups and enables per backup actions (Restore, Delete and Get).

### [Hosting site make](https://github.com/mglaman/hosting_site_make)

Allows a site to have its modules built from a makefile in the sites directory.

### [Hosting tasks extra](https://www.drupal.org/project/hosting_tasks_extra)

This module extends Aegir hostmaster (and drush/provision) with some additional tasks. Such as: cache-clear and registry-rebuild.
Now also includes the functionality from [Aegir HTTP basic authentication](https://github.com/computerminds/aegir_http_basic)

### [Hosting Variables](https://www.drupal.org/project/hosting_variables)

Allows you to set arbitrary custom Drupal variables for each site, such as site name and slogan.
These variables will be put in settings.php, and so can't be overriden (or changed) through the site interface.

### [Hosting Wordpress](https://www.drupal.org/project/hosting_wordpress)

Module to manage WordPress sites. It aims to support the main Aegir functionality, such as installation, upgrade, migration and backups.


## 2. Extensions to Provision (backend)

Starting from Aegir 7.x-3.x the Drush component can be included in a 'drush' directory in the same git repository as the hosting module.

Therefore this list will be shorter then for previous versions.

### [Provision STS](https://github.com/mlutfy/provision_sts)

Adds the Strict Transport Security header to hosts that require SSL.

## 3. Themes

*   [Saeven](https://www.drupal.org/project/saeven)
*   [Eldir](https://www.drupal.org/project/eldir). (The default theme)

## 4. Configuration management - e.g. Puppet modules and Chef cookbooks

Those projects allow you to manage Aegir instance(s) through a configaration management tool.

*   [Aegir Puppet module](https://www.drupal.org/project/puppet-aegir)
*   [Drush Puppet module](https://www.drupal.org/project/puppet-drush)
*   [Aegir3 Chef Cookbook](https://supermarket.chef.io/cookbooks/aegir3)
*   [Ansible role](https://github.com/GetValkyrie/ansible-role-aegir/)

## 5. Others

### [Aegir up](https://www.drupal.org/project/aegir-up)

Aegir-up is a Drush extension that deploys a local instance of the Aegir Hosting System atop Vagrant and Virtualbox, for development and testing purposes.

### [DevShop](https://www.drupal.org/project/devshop)

DevShop is a Drupal Development Environment Manager built on Aegir.

DevShop creates Aegir platforms and sites automatically from Git URLs. It tracks multiple Projects and allows multiple environments to be created for each Project, such as dev, test, and live. It provides tools to Pull Code, Sync Data, Commit Features, and Run Tests on these environments, and provides a dashboard with useful links and information for developers.

### [Aegir Pathologic Files](https://github.com/Lab43/aegir-pathologic-files)

A tiny Drupal Module to simplify file paths in content. This helps prevent broken images when the site directory name changes. Requires an apache rewrite rule to point /files to /sites/example.com/files, which Aegir provides by default.

This module is not for hostmaster, but for the sites hosted under Aegir.

## 6. Your module here?

_Developers:_ Please add your contributed module here. Pull requests welcome on https://github.com/aegir-project/documentation/
