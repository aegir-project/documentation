Release Process
===============

[TOC]

This page aims to document our release process. It documents the release cycle, but also the steps required to make a release.


The release cycle
-----------------

In general, each major Aegir release comprises a simultaneous release of all the modules that are part of the project. We generally go through several testing releases (alphas, betas & RCs) before doing the first stable release on a branch.

* First an **alpha** is released to test new functionalities and to accomplish the goals decided in the project roadmap for that major version. Example: `3.0-alpha1`
* When we have covered most of the functionalities outlined in the roadmap, we push out **beta releases** until no more critical issues show up. This is generally considered a _soft feature freeze_. Example: `3.0-beta1`
* Then we go into _full feature freeze_ and release a **first release candidate** (RC). Then a stable release branch is created, and the development branch is kept opened for development for the next stable release. This is generally considered a _soft API freeze_. Release candidates are made as long as critical bugs are found. Example: `3.0-rc1`.
* Once the development branch has no known critical bugs, the **stable release** is announced. From there on only critical fixes (security, critical performance and critical bugfixes) are committed to the stable branch, and stable releases are published (without alpha/beta/RC) directly on the stable branch. The stable branch is in **full API freeze**. New features are generally committed to the development branch.


Steps for a release
-------------------

### 1. Make sure our tests are all green

Look into [Jenkins](http://ci.aegirproject.org/) and [Travis](https://github.com/aegir-project/tests/) to see if all tasks have been performed without errors since the last commit. If there is an error, fix it before the release.

#### 1.1 Disable Debian dev builds

It is important to disable the Jenkins job named [D_aegir-debian-build-3x](http://ci.aegirproject.org/view/Debian%20dev%20builds/job/D_aegir-debian-build-3x/). This prevents later troubles in the [Publish the Debian packages](#10-publish-the-debian-packages) section. Otherwise a dev package may be built after the stable one, but before it has been promoted to the `testing` and `stable` repositories. As this will have a higher version number, it would take precedence when publishing to the `stable` repo.

### 2. Verify drupal-org.make

In the hostmaster project we maintain our own drupal-org.make file. Verify that drupal-org.make specifies up-to-date versions. Check that e.g. the ctools version specified is not out-dated.

### 3. Generating the release notes

We build complete release notes for every release. Those are made up of a summary of the release, an outline of key changes, of known issues, install and upgrade instructions and a full list of bugfixes and new features.
As we have a number of Drupal.org projects to cover we try to centralize our release notes combined with the documentation. Published under the release notes section.
On Drupal.org we add a link to the version specific release notes page on the release nodes for all projects we cover.

Using [Git Release Notes for Drush](https://www.drupal.org/project/grn)
 with [a patch for the MD format](https://www.drupal.org/node/2609134):

`drush rn --baseurl=https://www.drupal.org/ --md 7.x-3.3 HEAD`

or after placing the tags

`drush rn --baseurl=https://www.drupal.org/ --md 7.x-3.3 7.x-3.4`

This is done for the different git projects. Changing the first line to add 'to $name'.
The developers then proceed to format/edit the list of fixes as well as list other significant information/changes for this release. These notes end up becoming the Release Notes for the release.

Add it as a new file under [release-notes](https://github.com/aegir-project/documentation/tree/3.x/docs/release-notes), and add a line to [mkdocs.yml](https://github.com/aegir-project/documentation/blob/3.x/mkdocs.yml)

### 4. Running the release.sh script

Each time we make a new release, we run a script called `release.sh` in provision. This script does all the 'hard' work in that it doesn't forget all the very many places to edit version numbers etc of relevant documentation and other scripts. This includes install.sh.txt and upgrade.sh.txt.

Paraphrasing from the script itself:

```
$ ./release.sh 3.4

Aegir release script
====================

This script should only be used by the core dev team when doing an
official release. If you are not one of those people, you probably
shouldn't be running this.

This script is going to modify the configs and documentation to
release 7.x-3.7.

The following operations will be done:
 0. prompt you for a debian/changelog entry
 1. change the makefile to download tarball
 2. change the upgrade.sh.txt version
 3. display the resulting diff
 4. commit those changes to git
 5. lay down the tag
 6. revert the commit
 7. clone fresh copies of hosting/hostmaster and eldir to lay down the tag
 8. (optionally) push those changes
 9. clone fresh copies of golden contrib to lay down the tag
10. (optionally) push those changes

 ARE YOU SURE you disabled the D_aegir-debian-build-3x job in Jenkins?

The operation can be aborted before step 8. Don't forget that as
long as changes are not pushed upstream, this can all be reverted (see
git-reset(1) and git-revert(1) ).
```

Notice how we just provide the Aegir release number (`3.4`) to the release script, not the Drupal branch (`7.x`), which is hardcoded in the script to remove potential confusion.


### 5. Test the manual install in Jenkins

Before making a full release, test the release in Jenkins. To do so, start a build of the [P_Aegir_Puppet_Module_functional_test_Aegir3-dev-Drush8](http://ci.aegirproject.org/job/P_Aegir_Puppet_Module_functional_test_Aegir3-dev-Drush8/) job. Similar job exist for [Drush7](http://ci.aegirproject.org/job/P_Aegir_Puppet_Module_functional_test_Aegir3-dev-Drush7/) and [Drush6](http://ci.aegirproject.org/job/P_Aegir_Puppet_Module_functional_test_Aegir3-dev-Drush6/)

If any of these builds fail, delete the remote tags (using `git push origin :7.x-3.4`, for example), fix the bugs and start again.


### 6. Build the Debian packages

Build the package and upload to [http://debian.aegirproject.org/](http://debian.aegirproject.org/ "http://debian.aegirproject.org/"). Jenkins can build and upload a Debian package for you with [the S_aegir-debian-official-3.x job](http://ci.aegirproject.org/job/S_aegir-debian-official-3.x/). Enter the latest tag in the `JENKINS_AEGIR_TAG` field (e.g. `7.x-3.4`).

If you need to move the tags again, you will need to clear the testing archive using the [R clear repo job](http://ci.aegirproject.org/job/R%20clear%20repo/), with the testing argument.

You can also build and upload the package yourself as explained in these [detailed instructions](release-process/debian-packaging/). We first upload the package to the `testing` distribution, and it gets migrated down into `stable` after tests.


#### 6.1 Fix Debian packages only

When there's a bug in the Debian packaging itself we can do a minor package version update. **USE WITH CARE!**

    cd <provision source>
    <Fix>
    git commit
    git tag <previous version tag>.1
    git push --tags

TODO: We also need to update version numbers(as release.sh does), but only for provision. Maybe the script can get an option for it...

We don't create a release node on Drupal.org, just run the [S_aegir-debian-official-3.x job](http://ci.aegirproject.org/job/S_aegir-debian-official-3.x/) job. And publish as listed below.


### 7. Creating release nodes on Drupal.org

Once the tags are pushed and release notes published, we create a release node with an excerpt of (and a link to) the release notes so that tarballs are created and issue queue versions updated.

Using the template (change the version number):
```
See the full release notes at: http://docs.aegirproject.org/en/3.x/release-notes/3.7/
```

This needs to be done in the [provision](https://drupal.org/node/add/project-release/196005) and [hosting](https://drupal.org/node/add/project-release/196008) and (maybe) [eldir](https://drupal.org/node/add/project-release/452774) projects on Drupal.org.
And for Golden Contrib...
[hosting_civicrm](https://drupal.org/node/add/project-release/1982662),
[hosting_git](https://drupal.org/node/add/project-release/2217793),
[hosting_remote_import](https://drupal.org/node/add/project-release/1405640),
[hosting_site_backup_manager](https://drupal.org/node/add/project-release/1459830) and
[hosting_tasks_extra](https://drupal.org/node/add/project-release/1738498)

WAIT.... And after those are fully build... in the [hostmaster](https://drupal.org/node/add/project-release/195997) project.

Note: this could be [automated](https://www.drupal.org/node/1050618) with the right stuff on Drupal.org.

### 8. Test the upgrade in Jenkins

Once all release nodes have been created you can test the upgrade of the Debian packages by running the following Jenkins job:

* [7.x-3.x-stable-to-unstable](http://ci.aegirproject.org/view/Upgrades/job/U_aegir_7.x-3.x-stable-to-unstable-deb-package)
* [6.x-2.x_to_7.x-3.x_upgrade](http://ci.aegirproject.org/view/Upgrades/job/U_aegir_6.x-2.x_to_7.x-3.x_upgrade/)

### 9. Manually test install and upgrade

If the Jenkins tests are disabled in one of the early steps, you should test the install and upgrade to the new version in a local VM. (Vagrant is very useful for this, the provision repo has a Vagrantfile.)

At this point, the unstable repo actually contains the future stable version (ie 3.1 instead of 3.1-dev-abc). Check that it actually does: <http://debian.aegirproject.org/dists/unstable/main/binary-amd64/Packages>

To test the install, just use the instructions on http://www.aegirproject.org but replace "stable" by "unstable". You should end up with a stable version.

To test the upgrade, create a new VM, install normally with the aegirproject.org instructions, then replace stable by unstable in your lists (/etc/apt/), and run

    apt-get update && apt-get upgrade

It should ask you if you want to upgrade Aegir3. Say yes and make sure there are no errors.

If you don't encounter errors in this procedure, you're good to go.

#### 9.1. Quick test VMs

You can use an Vagrant base box to quickly test the install and upgrade. Some such boxes are optimized for Aegir development and testing:

* Debian VirtualBox: [PraxisLabs/jessie64-aegir3-dev.box](https://atlas.hashicorp.com/PraxisLabs/boxes/jessie64-aegir3-dev.box)

(Please add your base box to the list if relevant.)

### 10. Publish the Debian packages

Finally, when the Debian packages are tested you will need to pull them into the stable release channel:

We pull to stable (since the betas), manually:

    ssh jenkins@ci.aegirproject.org
    sudo su - reprepro
    reprepro@zeus:~$
    reprepro copy stable unstable aegir3
    reprepro copy stable unstable aegir3-hostmaster
    reprepro copy stable unstable aegir3-provision
    reprepro copy stable unstable aegir3-cluster-slave

If Jenkins has managed to build .debs and upload them before you've have a chance to pull them into testing/stable, you can manually remove them like so:

    reprepro remove unstable aegir3-cluster-slave aegir3 aegir3-provision aegir3-hostmaster

This should be prevented by disabling the Jenkins jobs named `D_aegir-debian*` before starting to push the release tags.

You can then re-upload the new .debs you've generated, using the '-f' (force) flag:

    dput -f aegir build-area/aegir2-provision_2.3_amd64.changes

After doing that, you can re-run the 'copy' commands to publish the .debs to the appropriate releases.


### 11. Publish the release notes widely

Once all this is done and the tarballs are generated, the release notes are published in:

*   The [docs](http://docs.aegirproject.org)
*   Update the upgrade.sh link in the [documentation](http://docs.aegirproject.org/en/3.x/install/upgrade/#upgrades-with-upgradesh-script)
*   The topic of the IRC channel
*   The aegir-announce mailing list ([announce@lists.aegirproject.org](mailto:announce@lists.aegirproject.org).)
*   Twitter as @aegirproject
    *   Template: "#aegir 3.? released! Release notes at http://docs.aegirproject.org/en/3.x/release-notes/3.?/"

Optionally, blog posts on [koumbit.org](http://koumbit.org), [mig5.net](http://mig5.net), and elsewhere may go into further detail about significant changes, screencasts etc.

### 12. Re-enable the debian dev build job

The job that was disabled in section 2.1.1 can now be enabled again.
