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
