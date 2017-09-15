Git Repositories
================

Aegir consists of a number of components each living in their own Git repository.

To make thinks more interresting (or complex) we also have these spread over [Drupal.org](https://www.drupal.org/project/hostmaster), [GitHub](https://github.com/aegir-project) and [GitLab.com](https://gitlab.com/aegir).

This document is intended to give some guidelines as to what should go where. It's intended for Aegir core, but we welcome any contributed project to follow. Just to have some consistency.


- Drupal.org is the canonical repository for the stable branches and releases.
- Issues should be on Drupal.org
- Feature branches can best be pushed to GitHub to facilitate testing with Travis.
- Pull Requests on GitHub are very welcome, please just add a link to them in an issue on Drupal.org.


### Branch naming

7.x-3.x is the main branch for all the core projects. This follows the Drupal default.

Feature branches can best be prefixed with 'feature/' resulting in 'feature/[issue number]-some-change'. Pushing them to GitHub and creating a PR for it lets us test the results with Travis.
Thease feature branches don't have to be pushed to D.o, and can be removed after being merged.

### Pull requests

- When a PR is not ready you can prefix the title with: `[WIP]`.

### Golden Contrib

We started adding more modules during the 3.x cycle and invented the *Golden Contrib* name for it.

The idea is to package modules that extend Aegir to make them easier to install.  [Their descriptions](http://docs.aegirproject.org/en/3.x/extend/contrib/#golden-contrib) are listed on the Contributed Projects page.

These are included from our [makefiles](http://cgit.drupalcode.org/hostmaster/tree/drupal-org.make) and end up in `profiles/hostmaster/modules/aegir/`.

#### Golden contrib module guidelines

- It should have a full release on Drupal.org
- It should be covered by the security team.
- There should be the intention to maintain the module for the longer term, at least as long as the current stable Aegir release.
- All Aegir core maintainers should have access to the module project on Drupal.org.  However, a minimum of two is required at all times. Use [the current maintainers list](http://docs.aegirproject.org/en/3.x/community/core-team/#current-members) for reference.
- Where appropriate it should include a file for Aegir's feature system. E.g. [hosting.feature.modulename.inc](http://cgit.drupalcode.org/hosting/tree/example/site_data/hosting.feature.site_data.inc)
- It should be security reviewed by other core maintainers, keeping the following in mind:
    * The module should not increase the system's attack surface.
    * etc.
* It should not encourage users to perform dangerous operations, from which there is no recovery.  If such features are requested, their use must come with warnings. 