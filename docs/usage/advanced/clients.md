Client management
=================

[TOC]

In some situations you may wish to allow different people access to Aegir, but restrict which sites they can manage.

* **Use Case 1:**
  As a developer you may simply wish to segregate your sites in Aegir by different clients for your own internal management reasons.
* **Use Case 2:**
  If you have many staff working on different projects, you may wish to issue them with different logins and restrict which sites they can access on the Aegir system.
* **Use Case 3:**
  As a drupal hosting company you might allow clients to sign up via your promotional website (using the 'Signup Form' feature in Aegir!), and then have their own account created on Aegir, with their newly provisioned site. They can then login and manage their own upgrades (by migrating to new platforms).  This functionality is provided by the Client feature in Aegir. Here's a brief guide to using it...

To grant clients secure access to the CLI, see the page on [Provision ACL](/usage/advanced/provisionacl/)

Enable the Clients feature
--------------------------

If you didn't enable this feature during installation, you can do so now by choosing the following from the menu bar at the very top of the page: **Hosting > Features**. On this page, click on 'Experimental' to expand these feature options. You can tick 'Clients' and click on the 'Save Configuration' button.

**NOTE:** Experimental features are designed to be for developer preview only, although we do have reports of them being successfully used in production environments - but do so at your own risk!


Configure the Client feature options
------------------------------------

From the top admin menu select **Hosting>Clients**. On this page you can configure how you want this feature to operate, with the following options:

**Automatically Create User Accounts for New Clients** -- If you tick this option, whenever you setup a new Client within Aegir, they will have a drupal user account created for them to be able to access their sites within the Aegir site. You will definitely want to leave this unticked for Use Case 1, perhaps also for Use Case 2, but may want to tick this for Use Case 3.

**Send Welcome Mail to New Clients** -- If this setting is ticked, the client will be sent an email when you create their drupal user account allowing them access to the system. Below this you can configure the email that will be sent.


Create clients
--------------

Creating a new Client on Aegir is as simple as selecting **Create Content>Client**. Then enter their email address (twice for confirmation), Name and Organization. Now click 'Save'.  *A [bug](https://www.drupal.org/node/2698687) was introduced in Aegir 3 preventing capture of email addresses and organization names. For now, it's only possible to enter client names.*

Depending on the options you selected above they might also have a user account created for them at this stage by Aegir, and then be emailed with their login details.

Otherwise the Client record will just be stored in the database and you will be able to assign sites to this client in the future.


Adding new users to a client
----------------------------

Perhaps you didn't initially allow clients to access their own Client area within Aegir and just kept it for your own internal use, or perhaps you now want to add additional users to a Client on the system.

To do this visit the Client page and select the 'Edit' tab. You'll see a list of allowed users. You can add users to this list by typing their username in the autocomplete field, or you can remove users by tciking the box next to their username and saving the form.

If the user you wish to add doesn't yet have a drupal user account on the Aegir site, you can create one as normal by going to the following on the top admin menu: **User Management>Users>Add User**. Then you can return to the client page and add their username to give them access to that client's area.


Clients accessing the system
----------------------------

If you have selected above that new Clients should also have an Aegir account setup for them, then they will be able to access the Aegir site.

Within Aegir they will ONLY be able to see details about their own sites.

