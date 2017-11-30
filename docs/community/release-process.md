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

Look into [GitLab CI](http://gitlab.com/aegir/provision/pipelines/) and [Travis](https://github.com/aegir-project/tests/) to see if all tasks have been performed without errors since the last commit. If there is an error, fix it before the release.


### 2. Verify drupal-org.make

In the hostmaster project we maintain our own drupal-org.make file. Verify that drupal-org.make specifies up-to-date versions. Check that e.g. the ctools version specified is not out-dated.

### 3. Generating the release notes

We build complete release notes for every release. Those are made up of a summary of the release, an outline of key changes, of known issues, install and upgrade instructions and a full list of bugfixes and new features.
As we have a number of Drupal.org projects to cover we try to centralize our release notes combined with the documentation. Published under the release notes section.
On Drupal.org we add a link to the version specific release notes page on the release nodes for all projects we cover.

Using [Git Release Notes for Drush](https://www.drupal.org/project/grn)
 with [a patch for the MD format](https://www.drupal.org/node/2609134):

`drush rn --baseurl=https://www.drupal.org/ --md 7.x-3.10 HEAD`

or after placing the tags

`drush rn --baseurl=https://www.drupal.org/ --md 7.x-3.3 7.x-3.4`

This is done for the different git projects. Changing the first line to add 'to $name'.
The [script release_notes.sh](http://cgit.drupalcode.org/provision/tree/scripts/release_notes.sh) can be used to automate this step.

The developers then proceed to format/edit the list of fixes as well as list other significant information/changes for this release. These notes end up becoming the Release Notes for the release.

Add it as a new file under [release-notes](https://github.com/aegir-project/documentation/tree/3.x/docs/release-notes), and add a line to [mkdocs.yml](https://github.com/aegir-project/documentation/blob/3.x/mkdocs.yml). And update the default page for the 'Release notes' section.

### 4. Running the release.sh script

Each time we make a new release, we run a script called `release.sh` in provision. This script does all the 'hard' work in that it doesn't forget all the very many places to edit version numbers etc of relevant documentation and other scripts. This includes updating upgrade.sh.txt.

It's best to start in a freshy cloned provision checkout. An extra remote to GitLab is needed for the packaging by GitLab CI.

```
git clone $USER@git.drupal.org:project/provision.git
cd provision/
git remote add gitlab git@gitlab.com:aegir/provision.git
```

Paraphrasing from the script itself:

```
$ ./scripts/release.sh 3.130 3.120

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
 3. change the .gitlab-ci.yml to build a release package
 4. display the resulting diff
 5. commit those changes to git
 6. lay down the tag
 7. revert the commit
 8. clone fresh copies of hosting/hostmaster and eldir to lay down the tag
 9. (optionally) push those changes
10. clone fresh copies of golden contrib to lay down the tag
11. (optionally) push those changes

The operation can be aborted before step 8. Don't forget that as
long as changes are not pushed upstream, this can all be reverted (see
git-reset(1) and git-revert(1) ).
```

Notice how we just provide the Aegir release number (`3.11`) to the release script, not the Drupal branch (`7.x`), which is hardcoded in the script to remove potential confusion.


### 5. Watch the GitLab CI pipelines

After the release script pushes it's commits you should give Ci some time to finish it's tests.

If any of these builds fail, one option might be to delete the remote tags (using `git push origin :7.x-3.4`, for example), fix the bugs and start again.



#### 6 Fix Debian packages only

See [Creating a Debian mini release](/community/release-process/debian-packaging/#creating-a-debian-mini-release)

### 7. Creating release nodes on Drupal.org

Once the tags are pushed and release notes published, we create a release node with an excerpt of (and a link to) the release notes so that tarballs are created and issue queue versions updated.

Using the template (change the version number):
```
See the full release notes at: http://docs.aegirproject.org/en/3.x/release-notes/3.11/
```

This needs to be done in the [provision](https://drupal.org/node/add/project-release/196005) and [hosting](https://drupal.org/node/add/project-release/196008) and [eldir](https://drupal.org/node/add/project-release/452774) projects on Drupal.org.
And for Golden Contrib...
[hosting_civicrm](https://drupal.org/node/add/project-release/1982662),
[hosting_git](https://drupal.org/node/add/project-release/2217793),
[hosting_logs](https://www.drupal.org/node/add/project-release/1871490),
[hosting_remote_import](https://drupal.org/node/add/project-release/1405640),
[hosting_site_backup_manager](https://drupal.org/node/add/project-release/1459830) and
[hosting_tasks_extra](https://drupal.org/node/add/project-release/1738498)

WAIT.... And only after those are fully build... in the [hostmaster](https://drupal.org/node/add/project-release/195997) project.

Build errors don't show up onde the release node page ... but on [the releases list]https://www.drupal.org/project/hostmaster/releases)

Note: this could be [automated](https://www.drupal.org/node/1050618) with the right stuff on Drupal.org.


### 9. Manually test install and upgrade

Just to make sure you should test the install and upgrade to the new version in a local VM. (Vagrant is very useful for this, the provision repo has a Vagrantfile.)

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

    ssh aegir0.aegir.coop
    sudo su - reprepro
    reprepro@aegir0:~$
    reprepro copy stable unstable aegir3
    reprepro copy stable unstable aegir3-hostmaster
    reprepro copy stable unstable aegir3-provision
    reprepro copy stable unstable aegir3-cluster-slave

If CI has managed to build .debs and upload them before you've have a chance to pull them into testing/stable, you can manually remove them like so:

    reprepro remove unstable aegir3-cluster-slave aegir3 aegir3-provision aegir3-hostmaster

You can then re-upload the new .debs you've generated, using the '-f' (force) flag:

    dput -f aegir build-area/aegir2-provision_2.3_amd64.changes

After doing that, you can re-run the 'copy' commands to publish the .debs to the appropriate releases.

### x. Push the revert commit

This is the final step in the release.sh script.


### 11. Create a specific Docker file

Create a new file like releases/Dockerfile-3.11 in the [dockerfiles](https://github.com/aegir-project/dockerfiles/) repo.
Then create a build for it on:
https://hub.docker.com/r/aegir/hostmaster/~/settings/automated-builds/


### 11. Publish the release notes widely

Once all this is done and the tarballs are generated, the release notes are published in:

*   The [docs](http://docs.aegirproject.org)
*   Update the upgrade.sh link in the [documentation](http://docs.aegirproject.org/en/3.x/install/upgrade/#upgrades-with-upgradesh-script)
*   The topic of the IRC channel
*   The aegir-announce mailing list ([announce@lists.aegirproject.org](mailto:announce@lists.aegirproject.org).)
*   Twitter as @aegirproject
    *   Template: "#aegir 3.? released! Release notes at http://docs.aegirproject.org/en/3.x/release-notes/3.?/"

Optionally, blog posts on [koumbit.org](http://koumbit.org), [mig5.net](http://mig5.net), and elsewhere may go into further detail about significant changes, screencasts etc.

### 12. Debian repository management

The repo is managed with a tool called reprepro.
Gitlab scp's to /var/www/repos/incoming/ on the repo server. It has an ssh key stored as a secret variable.

On the repo server incrond waits for new files and trigers reprepro using /usr/local/bin/reprepro_process_incomming for them.



Setting up a GitLab runner
==========================

GitLab.com offers the use of shares runner VM's to use for CI, but we need a priviledged one to be able to use sudo and sub docker images.
As a project we maintain one runner for this, but if you want to experiment more in a feature branch of your own fork then it's best to setup a new one.
Documentation on setting one up is on https://docs.gitlab.com/runner/install/
The minimal memory requirement seems to be 1GB. And up to 10GB of disk storage.

Example config file /etc/gitlab-runner/config.toml:
```
concurrent = 1
check_interval = 0

[[runners]]
  name = "Initfour-helmo-ci_runner01"
  url = "https://gitlab.com/ci"
  token = "9N-J78KC9b7oQE8LMP_d"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "ruby:2.1"
    privileged = true
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
  [runners.cache]
```

The modified lines are 'privileged' and 'volumes'


Troubleshooting CI tests
====


When you have your own GitLab CI runner you can inspect the container while it's running the tests.
Adding an extra `sleep 20m` as a last task in the .gitlab-ci.yml script section would keep it alive a bit longer. The default timeout is 3600 seconds.

Use `docker ps` to get the container ID, and `docker exec -ti [containerID] bash` to get a shell inside the container.

