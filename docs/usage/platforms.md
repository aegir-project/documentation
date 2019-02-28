Setting up a Platform
=====================

[TOC]

What is a platform anyway?
--------------------------

Platforms are a type of node in Aegir and often the source of confusion for new users. This is because the term or concept isn't really used explicitly outside of Aegir - Aegir is a system that suddenly makes Platforms 'make sense' to have.

The simplest definition of a Platform is a copy of Drupal core. That's really it. When you download a copy of Drupal from drupal.org, the result is what Aegir thinks of as a 'Platform'. No sites exist on it yet.

Before you can create a site using Aegir, you must first define the Platform.  This tells Aegir where to store the site directory, settings.php etc on the system.

In short: first you create a Platform, and a site 'lives' on that platform, in exactly this fashion:

* `/var/aegir/platforms/drupal-7.41` (Platform)
    * `/sites/example.com/settings.php` (site)

This abstraction is somewhat unique to Aegir in that it opens up a world of new opportunities for you. By managing Platforms or copies of Drupal core, and understanding what sites are on what copy of core, Aegir is capable of moving sites between platforms (which is effectively upgrading a site) - read more about this in Migrating/upgrading and renaming sites.

What else could be considered a platform?
-----------------------------------------

Anything that is more or less a copy of Drupal core is something Aegir considers a 'platform'. This thus includes any Drupal distribution, such as OpenAtrium, Pressflow, Acquia Drupal, OpenPublish, ManagingNews, and so on.

The key difference between such distributions and a standard 'vanilla' copy of Drupal core is that these distributions tend to have:

* a custom install profile in the /profiles/ directory
* a set of contrib or custom developed modules, libraries or themes shipped with the core or profile

If you build a copy of Drupal core and place your custom install profile in /profiles, this could be considered a Platform.

If you place your custom install profile in an existing Platform and re-Verify the Platform, that profile will now be recognised as an option when creating a site on that Platform.

Various 'versions' of Drupal, such as Drupal 6.x, Drupal 7.x or Drupal 8.x, are all considered separate Platforms in Aegir.

Is this paradigm clear? 'Sites' are managed inside 'Platforms'. 'Platforms' are managed inside 'Servers'. 'Servers' are managed by Aegir. In this sense, Aegir manages all the rest too.


Getting a Platform onto your server
-----------------------------------
You can create platforms on your server with either the web interface or the backend CLI.

### Prepare your codebase
Before you can add your platform via the web interface, you need to prepare your codebase in either a git repository or a drush make file.

If using a git repository, it must contain either a drush make file, a composer.json file, or an entire Drupal codebase. 


 - [Example makefiles](http://cgit.drupalcode.org/provision/tree/provision-tests/makes)
 - [Example git repository with drush make](https://github.com/aegir-project/example-drush-make)
 - [Example git repository with Drupal Composer](https://github.com/aegir-project/example-drupal-composer)

#### Prepare a Makefile platform

1. Create a drush makefile or find a drush make file online and copy it's URL.
2. Enter the URL of your makefile into the "Deploy from Makefile" section on the "Add Platform" page.
3. Press the "Save" button.
4. When the "Verify" task starts, it will run `drush make` on your makefile. If the makefile is invalid, or the URL is inaccessible, the task will fail. If the task fails, you can click **Edit** and change the makefile URL.
5. When the "Verify" task completes, you can click **Add Site**. See [Add Site Documentation]().

#### Prepare a Git repository platform

Create a git repository with your site's codebase in it. There are a few options for the structure of the repo. Choose one:

  1. You can add the entire Drupal stack to the codebase. You are then responsible for updating the code and committing to git.
  2. You can add a drush makefile to your git repository.
  3. You can add Drupal core and `composer.json` without the `vendor` folder. Aegir will run `composer install` for you.
  4. You can add the Drupal Composer project, which contains a `composer.json` file, and sets up the `web` folder as the document root.
  5. See [Drupal.org](https://www.drupal.org/docs/develop/using-composer/using-composer-to-manage-drupal-site-dependencies) for more documentation on using Composer and Drupal.
  
Remember the relative document root for your git repository and path to makefile, if you have one. 

When adding your repo to Aegir, you can use HTTPS or SSH syntax, but access must be granted to the `aegir` user on your system. If cloning an SSH repo, try to do it first using the command line interface to ensure Host Key checking is allowing your git host and SSH keys are properly authenticating."

Using Git with Aegir requires enabling at least one extra module: **Aegir Git**, [Aegir Deploy](https://www.drupal.org/project/hosting_deploy) or both.

To enable Aegir Git:

  1. Click **Hosting** in the Admin Toolbar at the top of the page.
  2. Click **Advanced** fieldset.
  3. Check **Git integration**.
  4. If you want to be able to run `git pull` from the web interface, check **Git pull task**.
  5. If you want to be able to run `git checkout` from the web interface to checkout a different branch or tag, check **Git Checkout task**.
  6. Press **Save Configuration**.

To enable Aegir Deploy:

  1. Read [the documentation](https://matteobrusa.github.io/md-styler/?url=cgit.drupalcode.org/hosting_deploy/plain/README.md).
  2. Click **Hosting** in the Admin Toolbar at the top of the page.
  3. Click **Experimental** fieldset.
  4. Click **Aegir Deployment**.
  5. If you want to deploy a Composer-managed platform, check **Composer Deploy via Git**.
  6. If you want to deploy a non-Composer-managed platform, check **Git Deploy (without Composer)**.
  7. Press **Save Configuration**.

### Adding a platform using the Hostmaster Web Interface 

Once you have your makefile or git repository ready, you can deploy Drupal using the web interface:

1. Log in to your Aegir Hostmaster site.
2. Click **Platforms** in the main navigation.
3. Click **Add Platform**.
4. Give your platform a **Name**. You can enter anything you would like, but it must be unique. By convention, you should include the project or distibution name and version, such as "MySite: Drupal 8.5.0".
5. The **Publish Path** the platform code will be deployed to is set automatically based on the platform **Name**. If you want to change it, click **Edit** next to **Publish Path**.
6. If using a makefile:
    1. Enter the full URL (or system path) to your makefile.
7. If using a git repository:
    1. Enter your git URL in **Repository URL**.
    2. Enter the relative path to docroot within your repository to **Repository Path**. (If also using a makefile, this path must not exist in the repo. This is where drush make will build your Drupal codebase.)
    3. If using a makefile, enter the absolute path to the makefile where it will be cloned to on the server. For example, if your **Publish Path** is `/var/aegir/platforms/drupal7`, and you have a file `drupal.make` in your git repository root folder, enter `/var/aegir/platforms/drupal7/drupal.make` into the field **Makefile**.
8. Press **Save**.

If everything was successful, you will have a clone of your git repository or a built copy of Drupal from your makefile.

If not, you can edit the platform to make changes and try again.

### Adding a platform using the Command Line Interface

A number of techniques exist to put a platform on a server. By convention, we
keep platforms in `/var/aegir/platform`. So that's a good place to begin:

    $ cd ~/platforms

#### Drush

For a standard Drupal platform, the easiest is to simply use drush:

    $ drush dl

#### Wget

You can also use the 'wget' command, for example:

    $ wget http://launchpad.net/prosepoint/trunk/0.23/+download/prosepoint-0.23.tar.gz
    $ tar -zxvf prosepoint-0.23.tar.gz

#### Version control

You could use Git, e.g.:

    $ git clone https://github.com/pressflow/7.git pressflow-7

#### Manually

You could download it on your PC and scp or FTP the files up onto your server.

#### Drush Make

Drush Make comes installed on your Aegir system by default. You can use Drush Make to 'build' a platform, which does the job of fetching core and any other contributed or custom packages that you specify in a makefile.

You can even specify the URL or path to a makefile in the form you are given when adding a Platform node in the Aegir frontend, and Aegir will execute the Drush Make command in the backend and build it for you!

Explaining how to use Drush Make is outside the scope of this document. Consult the README.txt of Drush Make to learn the makefile syntax, or use a ready-made makefile available from the web. Some distributions such as OpenAtrium provide an example makefile for you to build the distribution with.


#### Composer

When verifying a platform, Aegir runs `composer install` if a `composer.json` file is found. See [Drupal.org](https://www.drupal.org/docs/develop/using-composer/using-composer-to-manage-drupal-site-dependencies) for more information on using Composer to mananage Drupal.

##### Drush Options

There are a few drush options you can use to control composer install behavior:

  - `provision_composer_install_platforms` - Set to FALSE to prevent provision from ever running `composer install`.    Default is TRUE.
  - `provision_composer_install_platforms_verify_always` - By default, provision will run `composer install` every time a platform is verified. Set to FALSE to only run `composer install` once. If composer.json    changes, you will have to run `composer install` manually. Default is TRUE.
  - `provision_composer_install_command` - The full command to run during platform verify. Default is `composer install --no-interaction --no-progress --no-dev`.


Platforms and remote web servers
--------------------------------

Aegir has a 'spoke' model when it comes to remote servers, whereby the 'master' Aegir server keeps a copy of all platforms and sites and syncs changes outbound to remote servers that are running those platforms or sites, on tasks such as Verify etc.

Because of this, all platform names and paths must be unique, even across remote servers. This means you cannot have 'Drupal 7.0' in /var/aegir/platforms/drupal-7.0 on both Server A and Server B, because it can only exist once in that location on the master Aegir server. Platforms can't share the same name 'Drupal 7.0' because Aegir uses the platform name to define a 'context' by which it can refer to that server, and there can be no conflicts.

The exception to the unique path rule is when using web clustering (a collection of web servers running a platform), but even then, the platform is attached to the 'cluster' server and so is still 'unique' in this sense, that path cannot be reused for another platform running on another server somewhere else.

So when adding platforms to your filesystem and to Aegir, make sure that the platform is unique in name and path, so that other servers cannot try and use this reserved name/path for other platforms.

More on the platform name space

Migrating a site to a remote server illustrates this name space issue further: as an example; given that an Aegir hostmaster has two directories representing two platforms (and remembering that aegir does not migrate platforms, only sites):

* /var/aegir/platforms/plat-local  
* /var/aegir/platforms/plat-remote  

and the remote server also has:

* /var/aegir/platforms/plat-remote

then having aegir hostmaster migrate a site from platform plat-local to plat-remote will result in the site files being:

1. moved from .../plat-local to hostmaster's local .../plat-remote and
2. rsync'd to .../plat-remote on the remote server.

It is important to understand this if you are importing a working Drupal site into Aegir's control and Aegir hostmaster is on a different server from the working Drupal site.


Do I have to have multiple platforms?
-------------------------------------

No, you can simply have one platform or copy of Drupal core and provision all your sites to sit on the one platform. However, eventually you will want to upgrade your sites to a new copy of Drupal core, and rather than replace your core files in-place, it's recommended to build a new Platform with the newer copy of core, and use the Migrate task to move your sites (upgrade) onto the new code.


Adding the Platform node
------------------------

Now that you've got your Platform on your server, or you know where and how you're going to do it (say, with Drush Make), it's time to tell Aegir about your new Platform.

To do so, add a new node of type 'Platform' in your Aegir frontend. Typically this is most easily done by visiting Create Content > Platform in the Admin Menu.

The Platform node form has several fields required for giving Aegir information about your platform. These are:

* **Name** -- a descriptive name for your platform. You very likely want this to be something like "Drupal 7.41".
* **Publish Path** -- the path on the filesystem where the platform is, or will be when Drush Make builds it. This must be the absolute path, for instance `/var/aegir/drupal-7.41`
* **Makefile** -- the path on the filesystem to a makefile that will be used to create the platform.

Once you have completed entering this information, you can click the Save button. A 'Verify' task will be spawned and added to the Task queue (visible in the right sidebar). The backend will then parse this new platform and build up a registry of information about it, such as what version of Drupal it is, as well as what versions of install profiles, modules and themes are present on that platform.

Certain system configurations, such as Apache configurations similar to the .htaccess file that comes with Drupal, will be written to the filesystem, permissions checked and adjusted where necessary, and services restarted.

Now that you have a platform or codebase that Aegir is aware of, you can now proceed to install or import sites onto that platform!
