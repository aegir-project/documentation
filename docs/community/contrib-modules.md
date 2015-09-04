## 1. Extensions to Hostmaster (frontend)

### [Hosting Git](http://drupal.org/project/hosting_git)

This is a simple module for the Aegir project that adds a 'Git pull', 'Git checkout' and 'Git clone' task for sites.
The successor of Hosting site Git & Hosting platform Git

### [Hosting Logs](http://drupal.org/project/hosting_logs)

This is a simple module for the Aegir project that adds a 'Logs' tab to sites and platforms. Showing Apache error, Git commit and watchdog logs.

### [Hosting Platform Pathauto](http://drupal.org/project/hosting_platform_pathauto)

This is an add-on module for the Aegir hosting system: [http://aegirproject.org/](http://aegirproject.org/ "http://aegirproject.org/") This module simply makes a little time-saving tweak when creating a new platform: ...

### [Hosting site make](https://github.com/mglaman/hosting_site_make)

Allows a site to have its modules built from a makefile in the sites directory.

### [Hosting tasks extra](http://drupal.org/project/hosting_tasks_extra)

This module extends Aegir hostmaster (and drush/provision) with some additional tasks. Such as: cache-clear and registry-rebuild.
Now also includes the functionality from [Aegir HTTP basic authentication](https://github.com/computerminds/aegir_http_basic)

## 2. Extensions to Provision (backend)

Starting from Aegir 7.x-3.x the Drush component can be included in a 'drush' directory in the same git repository as the hosting module.

Therefore this list will be shorter then for previous versions.

### [Provision STS](https://github.com/mlutfy/provision_sts)

Adds the Strict Transport Security header to hosts that require SSL.

## 3. Themes

*   [Saeven](https://www.drupal.org/project/saeven)
*   [Eldir](http://drupal.org/project/eldir). (The default theme)

## 4. Configuration management - e.g. Puppet modules and Chef cookbooks

Those projects allow you to manage Aegir instance(s) through a configaration management tool.

*   [Aegir Puppet module](http://drupal.org/project/puppet-aegir)
*   [Drush Puppet module](http://drupal.org/project/puppet-drush)
*   [Aegir3 Chef Cookbook](https://supermarket.chef.io/cookbooks/aegir3)
*   [Ansible role](https://github.com/GetValkyrie/ansible-role-aegir/)

## 5. Others

### [Aegir up](http://drupal.org/project/aegir-up)

Aegir-up is a Drush extension that deploys a local instance of the Aegir Hosting System atop Vagrant and Virtualbox, for development and testing purposes.

### [DevShop](http://drupal.org/project/devshop)

DevShop is a Drupal Development Environment Manager built on Aegir.

DevShop creates Aegir platforms and sites automatically from Git URLs. It tracks multiple Projects and allows multiple environments to be created for each Project, such as dev, test, and live. It provides tools to Pull Code, Sync Data, Commit Features, and Run Tests on these environments, and provides a dashboard with useful links and information for developers.

### [Aegir Pathologic Files](https://github.com/Lab43/aegir-pathologic-files)

A tiny Drupal Module to simplify file paths in content. This helps prevent broken images when the site directory name changes. Requires an apache rewrite rule to point /files to /sites/example.com/files, which Aegir provides by default.

This module is not for hostmaster, but for the sites hosted under Aegir.

## 6. Your module here?

_Developers:_ Please add your contributed module here. Pull requests welcome on https://github.com/aegir-project/documentation/
