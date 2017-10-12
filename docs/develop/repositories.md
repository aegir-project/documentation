Git Repositories
================

Aegir consists of a number of components each living in their own Git repository.

To make things more interresting (or complex) we also have these spread over [Drupal.org](https://www.drupal.org/project/hostmaster), [GitHub](https://github.com/aegir-project) and [GitLab.com](https://gitlab.com/aegir).

This document is intended to give some guidelines as to what should go where. It's intended for Aegir core, but we welcome any contributed project to follow. Just to have some consistency.


- Drupal.org is the canonical repository for the stable branches and releases.
- Issues should be on Drupal.org
- Feature branches can best be pushed to GitHub or GitLab to facilitate testing with Travis or GitLab CI.
- Pull Requests on GitHub are very welcome, please just add a link to them in an issue on Drupal.org.


### Branches

#### Primary branches


* 3.x - current stable Drupal 7 frontend, Drush and provision. (maintainer: helmo)

* 4.x - The next development step, to [go beyond Drush](https://www.drupal.org/node/2912579) (maintainer: jon_pugh)

* 5.x - Aegir NG, [Drupal 8, new queue and modernized backend](https://www.drupal.org/node/2714641) (maintainers: ergonlogic, colan)


All core maintainers can work on these branches, however the noted maintainer for each branch is often the first to contact and the most involved in that particular area.


#### Branch naming


For the current stable release we have 7.x-3.x as the main branch for all the core projects. This follows the Drupal 7 default.

Feature branches can best be prefixed with 'feature/' resulting in 'feature/[issue number]-some-change'. Pushing them to GitHub and creating a PR for it lets us test the results with Travis.
These feature branches don't have to be pushed to D.o, and can be removed after being merged.


### Pull requests

- When a PR is not ready you can prefix the title with: `[WIP]`.
