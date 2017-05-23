Managing Your Site With Git
===========================

Using the [Aegir Hosting Git](https://www.drupal.org/project/hosting_git) module you can manage your site with Git within Aegir. This reduces the need to SSH/SFTP into your Aegir server every time you need to make a change, reducing the risk of something going wrong.

Within this guide, you will need to install the Aegir Git module on your Aegir instance.

Getting Started
===========================

### Step 1

Once you've setup your platform (which can also be managed by Git), you'll need to create a Git repository with the following structure:

    libraries/
    modules/
    themes/
    .gitignore
    versioned.settings.php (optional, you have to include it yourself from local.settings.php)
    README.txt (again optional, but good practice)

Within the `.gitignore` file add the following (this is just a suggestion):

    *~
    files
    private
    settings.php
    local.settings.php
    drushrc.php

Top tip: one trick to 'safely' store some `$conf` settings in Git is to include a file called `versioned.settings.php` from the `local.settings.php`.

### Step 2

Once you have committed your sites code you will need to log into Aegir and click create site on the platform you want to install the site to.  When creating the site enter your Git URL (if you have not done so yet add your service SSH key to your Git repository) and Aegir will give you some options as to how to trigger `Git Pull`.

### Step 3

On the site node's edit page you now have a field 'Pull Trigger URL' when you set the 'Git pull method' field to match yiou can add this url as webhook url in your Git repository manager.
