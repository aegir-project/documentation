Importing sites
===============

It is possible to import existing sites into the Aegir hosting system, so that they can be managed by Aegir.

These instructions assume you've set up a new server with Aegir on it, and you want to import sites into Aegir from another server, or even from the same server, using backups.

There are three standard ways of importing sites into Aegir:

* from [Aegir backups](import/aegir-backups)Aegir backups - simplest and fastest, scriptable
* from any [single site](import/single-site) manually - most reliable, can import even non-aegir sites, hard to automate
* from a [complete drupal install](import/complete-drupal-install) - fairly reliable, can import non-aegir sites, could be automated but requires access to the database servers currently in use by the site
