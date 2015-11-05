[Release process](/content/release-process)

*   [Developer](/category/level/developer "This level is generally for users interested in further developing the Aegir system itself, or extending it with contributed modules.")

This page aims to document our release process. It documents the release cycle, but also the steps required to make a release.

# 1. The release cycle

In general, each major Aegir release comprises a simultaneous release of all the modules that are part of the project. We generally go through several testing releases (alphas, betas & RCs) before doing the first stable release on a branch.

*   First an **alpha** is released to test new functionalities and to accomplish the goals decided in the [Project Roadmap](/node/35) for that major version. Example: `0.4-alpha1`, [`0.4-alpha15`](/0.4-alpha15)
*   When we have covered most of the functionalities outlined in the [roadmap](/roadmap), we push out **beta releases** until no more critical issues show up. This is generally considered a _soft feature freeze_. Example: [`0.4-beta1`](/0.4-beta1)
*   Then we go into _full feature freeze_ and release a **first release candidate** (RC). Then a stable release branch is created, and the development branch is kept opened for development for the next stable release. This is generally considered a _soft API freeze_. Release candidates are made as long as critical bugs are found. Example: [`0.4-rc1`](/0.4-rc1), [freeze announcement](/node/360)
*   Once the development branch has no known critical bugs, the **stable release** is announced. From there on only critical fixes (security, critical performance and critical bugfixes) are committed to the stable branch, and stable releases are published (without alpha/beta/RC) directly on the stable branch. The stable branch is in **full API freeze**. New features are generally committed to the development branch.

See also [the tag and branch naming convention](/content/branch-and-tagging-conventions).


# 2. Steps for a release

For the specifics of release naming conventions and the cycle, see [the branch naming convention](/node/187).


## 2.1. Make sure Jenkins is all green

Look into [Jenkins](http://ci.aegirproject.org/) to see if all tasks have been performed without errors since the last commit. If there is an error, fix it before the release.

### 2.1.1 Disable Debian dev builds in Jenkins

It's probably a good idea to disable the Jenkins jobs named 'D_aegir-debian*'. This prevents later troubles in the 'Publish the Debian packages' section.
The dev package is build before the stable one, and as 3.1+103.0544212 > 3.1 it gets added to the stable repo.

## 2.2. Verify that drupal-org.make specifies up-to-date versions

In the hostmaster project we maintain our own drupal-org.make file. Check that e.g. the ctools version specified is not out-dated.


## 2.3. Generating the release notes

We build complete release notes for every release. Those are made up of a summary of the release, an outline of key changes, of known issues, install and upgrade instructions and a full list of bugfixes and new features.

Using [Git Release Notes for Drush](https://www.drupal.org/project/grn)

`drush rn 7.x-3.0-beta1 HEAD`

or after placing the tags

`drush rn 7.x-3.0-alpha1 7.x-3.0-alpha2`

This is done for the different git projects. Changing the first line to add 'to $name'.
The developers then proceed to format/edit the list of fixes as well as list other significant information/changes for this release. These notes end up becoming the Release Notes for the release.

## 2.4. Running the release.sh script

Each time we make a new release, we run a script called `release.sh` in provision.

This script should only be used by the core dev team when doing an official release. If you are not one of those people, you probably shouldn't be running this.

This script does all the 'hard' work in that it doesn't forget all the very many places to edit version numbers etc of relevant documentation and other scripts. This includes install.sh.txt and upgrade.sh.txt.

Paraphrasing from the script itself:

```
The following operations will be done:
 0. prompt you for a debian/changelog entry  
 1. change the makefile to download tarball  
 2. change the upgrade.sh.txt version  
 3. display the resulting diff  
 4. commit those changes to git  
 5. lay down the tag  
 6. revert the commit  
 7. (optionally) push those changes  

The operation can be aborted before step 7. Don't forget that as  
long as changes are not pushed upstream, this can all be reverted (see  
git-reset(1) and git-revert(1) ).
```

Notice how we just provide the Aegir release number (`1.10`) to the release script, not the Drupal branch (`6.x`), which is hardcoded in the script to remove potential confusion.


## 2.5. Test the manual install in Jenkins

Before making a full release, test the release in Jenkins. To do so, start a build of the launch the [P_Aegir_Puppet_Module_functional_test_Aegir3-dev](http://ci.aegirproject.org/job/P_Aegir_Puppet_Module_functional_test_Aegir3-dev/) with the following parameters:

*   the right release as the `AEGIR_VERSION` parameter
*   the `DRUSH_VERSION` to match what is required for this release.

If the build fails, delete the remote tags (using `git push origin :6.x-1.7`, for example), fix the bugs and start again.


## 2.6. Build the Debian packages

Build the package and upload to [http://debian.aegirproject.org/](http://debian.aegirproject.org/ "http://debian.aegirproject.org/"). Jenkins can build and upload a Debian package for you with [the S_aegir-debian-official-3.x job](http://ci.aegirproject.org/job/S_aegir-debian-official-3.x/).

If you need to move the tags again, you will need to clear the testing archive using the [R clear repo job](http://ci.aegirproject.org/job/R%20clear%20repo/), with the testing argument.

You can also build and upload the package yourself as explained in these [detailed instructions](http://community.aegirproject.org/node/543). We first upload the package to the `testing` distribution, and it gets migrated down into `stable` after tests.

**Special, for 2.x**: build the package manually, see [detailed instructions](http://community.aegirproject.org/node/543). It should be something like:

    ./release.sh 2.0-rc5
    git reset --hard 6.x-2.0-rc5
    git-buildpackage -kanarcat@koumbit.org --git-builder=debuild
    dput aegir build-area/aegir2-provision_2.0~rc5_amd64.changes

See the [detailed instructions](http://community.aegirproject.org/node/543) for the dput configuration.

### 2.6.1 Fix Debian packages only

When there's a bug in the Debian packaging itself we can do a minor package version update. USE WITH CARE!

    cd <provision source>
    <Fix>
    git commit
    git tag <previous version tag>.1
    git push --tags

We don't create a release node on Drupal.org, just run the [S_aegir-debian-official-3.x job](http://ci.aegirproject.org/job/S_aegir-debian-official-3.x/) job. And publish as listed below.



## 2.7. Test the upgrade in Jenkins

Once both of those tasks have executed successfully, you can test the upgrade of the Debian packages by running the following Jenkins job:

[http://ci.aegirproject.org/view/Upgrades/job/U%20aegir%206.x-1.x%20deb%2...](http://ci.aegirproject.org/view/Upgrades/job/U%20aegir%206.x-1.x%20deb%20upgrade/ "http://ci.aegirproject.org/view/Upgrades/job/U%20aegir%206.x-1.x%20deb%20upgrade/")

(Note that this test is currently non-functional)


## 2.8. Creating release nodes on Drupal.org

Once the tags are pushed and release notes published, we create a release node with an excerpt of (and a link to) the release notes so that tarballs are created and issue queue versions updated.

This needs to be done in the [provision](https://drupal.org/node/add/project-release/196005) and [hosting](https://drupal.org/node/add/project-release/196008) and (maybe) [eldir](https://drupal.org/node/add/project-release/452774) projects on Drupal.org.
And after those are fully build in the [hostmaster](https://drupal.org/node/add/project-release/195997) project.

Note: this could be [automated](https://www.drupal.org/node/1050618) with the right stuff on Drupal.org.



## 2.10. Publish the Debian packages

Finally, when the Debian packages are tested you will need to pull them into the stable release channel:

We pull to stable (since the betas), manually:

    reprepro@zeus:~$
    reprepro copy stable unstable aegir3
    reprepro copy stable unstable aegir3-hostmaster
    reprepro copy stable unstable aegir3-provision
    reprepro copy stable unstable aegir3-cluster-slave

If Jenkins has managed to build .debs and upload them before you've have a chance to pull them into testing/stable, you can manually remove them like so:

    reprepro remove unstable aegir2-cluster-slave aegir2 aegir2-provision aegir2-hostmaster

This should be prevented by disabling the Jenkins jobs named 'D_aegir-debian*' before starting to push the release tags.

You can then re-upload the new .debs you've generated, using the '-f' (force) flag:

    dput -f aegir build-area/aegir2-provision_2.3_amd64.changes

After doing that, you can re-run the 'copy' commands to publish the .debs to the appropriate releases.


## 2.11. Publish the release notes widely

Once all this is done and the tarballs are generated, the release notes are published in:

*   The [docs](http://aegir.readthedocs.org/en/3.x/release-notes/3.1-release-notes/)
*   The topic of the IRC channel
*   The aegir-announce mailing list ([announce@lists.aegirproject.org](mailto:announce@lists.aegirproject.org).)
*   Twitter as @aegirproject

Optionally, blog posts on [koumbit.org](http://koumbit.org), [mig5.net](http://mig5.net), and elsewhere may go into further detail about significant changes, screencasts etc.
