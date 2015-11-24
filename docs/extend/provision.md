Provision - the Aegir backend
=============================

[TOC]

Each of the major entities managed by Aegir (namely servers, platforms and sites) are represented by nodes in the front end, which in turn generate Drush 'named contexts' in the backend. Named contexts are similar to what Drush refers to as 'site aliases', but are more structured and cover more than just sites.

Whenever you create or edit a node on the front end, Aegir will create or update the named context on the backend, and any tasks run on the entity will be run on that named context, ie:

    $ drush @server_master provision-verify

All of the actual work related to managing your system is done by the backend, and the front end primarily serves as a way to manage these aliases. You can very easily use the backend on it's own if you so choose.


Creating or updating named contexts
-----------------------------------

Contexts are created and updated through the provision-save command. The format of the command is:

    $ drush provision-save @contextname --context_type=type --property=value --other_property=value

Every time this command is called, it will update the named context and only modify the properties that have been specified. It is important to note that the provision-save command never has a site alias or context name before the command, but requires you to specify which context you are modifying as the first argument.


Deleting named contexts
-----------------------

To delete an existing named context, you simply need to pass the '--delete' option to the provision-save command, i.e.:

    $ drush provision-save @contextname --delete


Using named contexts
--------------------

Named contexts are used in 3 ways in provision: as aliases, arguments or options:

* **Aliases**: Aliases appear to the left of the drush command, and are making use of the Drush 'site alias' functionality.
```
    # Verify the site represented by @example.com
    $ drush @example.com provision-verify
```
* **Options**: Certain options to provision-save require you to pass the alias of another named context of a specific type, and uses this information to build relationships between objects.
```
    # generate a new site context in the directory represented by @platform_test
    $ drush provision-save @example.com --context_type=site --uri=example.com --platform=@platform_example
```
* **Arguments**: A select few provision commands require you to pass the alias of another specific type of named context as an argument.
```
    # Migrate the site to the platform represented by @platform_newversion
    $ drush @example.com provision-migrate @platform_newversion
```


Context types
-------------

There are (currently) 3 different entities represented as named contexts in the aegir backend. You determine which type of entity you are working with by specifying the context_type option to the provision-save command. You dont need to specify it every time, but it is best to always explicitly specify it, just in case.

Because the namespace of all contexts is shared, they each have different naming conventions to avoid collisions. There are certain named contexts of each type which are always present in the system.

Each of the context types have different semantics and properties, and certain provision commands will only operate on certain types of entities.

### Server contexts

**Naming guideline**: Server with a hostname of server.example.com becomes @server_serverexamplecom.

**Special contexts**: @server_master always refers to the local server where aegir is installed.

**Required properties**:

* *remote_host*: This should always be set to the FQDN of the server being managed.

**Services**: Servers are the only entities which have services associated to them. Services are things such as http, db or dns.

Each service may select one service_type, which chooses which implementation to load for this service. Examples of these are the 'mysql' implementation of the 'db' service, or the 'apache' and 'apache_ssl' implementations of the http service.

All services that are found by Aegir are enabled, but unless you specifically select one of the implementations, the 'null' service type is loaded. Each service type has additional required properties and values that can be passed to it, but describing them all is outside of the scope of this document.

**Important commands**:

* *provision-verify*: Verify that the configuration is correct and fix them if they aren't.
```
    # Enable a new remote server hosting sites with apache on port 8080
    $ drush provision-save @server_webexamplecom --context_type=server \
    --remote_host=web.example.com \
    --http_service_type='apache' \
    --http_port=8080

    # Verify that the settings are correct
    $ drush @server_webexamplecom provision-verify --debug
```

### Platform contexts

Platforms are how aegir refers to a specific copy of Drupal checked out on a file system. Platforms are hosted on web servers and sites are hosted on platforms.

**Naming guideline**: A platform in /var/aegir/codebase-6.1 becomes @platform_codebase61 , but this is not strictly enforced. We only use that naming convention when we do not have a node representing the platform available. If a node is available, we use the title of the node to generate the context name.

Basically, you can name your platform anything but it should still be prefixed with 'platform\_'.

**Special contexts**: There are no special contexts per se, but the hostmaster-install and hostmaster-migrate commands generate aliases for the hostmaster platform in the form of '@platform_hostmaster$version'. If the front end is present, one or more of these contexts are usually available too.

**Required properties**:

* *root*: The absolute path to a Drupal directory, generally in the form * `/var/aegir/platformdir`.

**Optional properties**:

* *web_server*: The name of a server context that provides the http service. The default is to @server_master makefile : The absolute path or url to a drush make makefile. If the root directory does not exist and the makefile property does, provision will call drush_make to attempt to create the directory.

**Important commands**:

* *provision-verify*: This will ensure that all the files are the correct permissions, and are all present on the correct remote servers. Also rebuild package database. Run this command whenever the contents of the platform changes.
```
    # New open atrium platform in /var/aegir/atrium-head that will be built from a
    # remotely hosted
    # makefile and published on an external web server.
    $ drush provision-save @platform_atriumhead --context_type=platform \
    --root=/var/aegir/atrium-head \
    --web_server=@server_webexamplecom \
    --makefile='http://omega8.cc/dev/atrium_stub.make.txt'

    # Verify settings and generate the directory with drush make, pushing to the
    # remote server after.
    $ drush provision-verify @platform_atriumhead
```

### Site contexts

**Naming guidelines**: A site with the url 'site1.example.com' becomes @site1.example.com. The context name is always the url.

**Special contexts**: @hostmaster and @hm always refer to the site running the hostmaster front end. This helps with debugging and makes it more easily scriptable.

**Required properties**:

* *uri*: The FQDN of the site. should match the context name.
* *platform*: The name of a platform context that the site will be hosted on.

**Optional properties**:

* *db_server*: The name of a server context which provides the db server. defaults to @server_master.
* *client_email*: The email address to send the login information for the admin user.
* *profile*: The installation profile the site is running. Dependent on the platform supporting it.
* *language*: The ISO language code of the site. Dependent on the platform and profile supporting it.

**Important commands**:

Almost all of the provision commands are related to sites, so this is just the very core of them. Check Drush help for more.

* *provision-install*: Install the site represented by the context. Just creating the context doesn't install the site yet.
```
    # Set up a new site to be installed on the open atrium platform (hosted on a
    # remote server), using an external database server, with the openatrium profile
    # and localize it into spanish.
    $ drush provision-save @atrium.example.com
    --context_type=site \
    --uri=atrium.example.com \
    --platform=@platform_atriumhead \
    --db_server=@server_dbexamplecom \
    --client_email=client@example.com \
    --profile=openatrium \
    --language=es

    # Install the new site to the specifications requested above.
    drush @atrium.example.com provision-install --debug
```
* *provision-verify*: Verify the site is running correctly, and ensure the files are all published to the correct servers.
```
    # Verify the site to make sure everything is ok.
    $drush @atrium.example.com provision-verify --debug
```
* *provision-delete*: Delete the site and all it's information. The context will still be there.


Importing contexts into the front end
-------------------------------------

Hostmaster provides a drush command which will attempt to generate or updates front end nodes that match your defined contexts. To import a context into the front end, just run :

    $ drush @hostmaster hosting-import @contextname

This will import that context, and every context it depends on into the front end. This is useful for integration with other systems such as the puppet-aegir module for Puppet, which can add newly configured servers directly to the front end.
