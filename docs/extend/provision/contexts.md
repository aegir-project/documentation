Provision Contexts
==================

[TOC]


Purpose
-------

Contexts are simply a way for Aegir to store certain pieces of information that it thinks may be useful later. They are named so that they can be referred to easily.

Every server, platform and site in Aegir has an associated context. Each one of these contexts stores information about the associated entity. So, for example, a site context stores the base URI of the site. Later, the code responsible for informing the web server of the site's existence can look at this context and determine the requested URI.

In theory contexts allow most of the back-end commands in Aegir to operate just on the name of the context. This is an important concept: the alternative would be to try to guess everything that the command being invoked might want to know and pass it in as a series of command line arguments.

For example, the command that Aegir uses to clone a site just needs to be given the context of the site to clone and the platform to clone it to. From there the code can work out where the destination platform actually is, what servers it needs to talk to, and with what credentials.

### Drush contexts

Note that Drush also has the concept of a context, but this is not to be confused with Aegir contexts which are different. Aegir contexts are closer to, and stored as, Drush 'aliases'.


Implementation
--------------

Contexts are always named to start with a '@' symbol, except for in the filename of the file in which they are stored.

Contexts in Aegir are stored as a simple array of data in a file within the backend of Aegir. However this array of data is accessed and modified with a set of objects and accessors. Here is an example of what a context file looks like:

    <?php
    $aliases['hostmaster'] = array (
      'context_type' => 'site',
      'platform' => '@platform_hostmaster',
      'server' => '@server_master',
      'db_server' => '@server_localhost',
      'uri' => 'aegir.example.com',
      'root' => '/var/aegir/hostmaster-0.4-beta2',
      'site_path' => '/var/aegir/hostmaster-0.4-beta2/sites/aegir.example.com',
      'site_enabled' => true,
      'language' => 'en',
      'client_email' => 'aegir@example.com',
      'aliases' => array (),
      'redirection' => false,
      'profile' => 'hostmaster',
    );

These files are stored in the `/var/aegir/.drush` directory on the master Aegir server.

In the above example we can see the properties of the context, like the 'uri' of the site, and the 'profile' that the site is running.

There are also a number of more interesting properties in the example, note the 'db_server' property, which has a value that is another context. We shall see later that these properties are special, and allow developers to easily access functions of the 'db_server' without needing to care which db server they are talking to.

Contexts are used within Drush commands as objects, which subclass provision_context. This allows much more flexibility and cleverness, though can make them very confusing to use sometimes! As a developer you will only need to worry about the context objects, as Aegir handles storing them in the files for you. But it is important to note that each context must be representable in a flat text file (or be prepared to do some serious leg-work) by that I mean you probably don't want to be storing massive amounts of relational data in them, use a database for that!


Using contexts in your code
---------------------------

Getting the current context is really easy, just call the 'd' accessor:

    <?php
      $current_context = d();
    ?>

What the current context is will depend on the drush command that is running and the context that was passed into that drush command. For example if you ran a site verify task, then the caller would name a site context, and thus initially calls to d() would return the site context. The values of the context are accessible as simple properties of this object.  So, suppose a verify task is started for the main Aegir frontend, which has a context that is always called '@hostmaster', the drush command invocation would look like this:

    $ drush @hostmaster provision-verify

Then the drush command for provision-verify could do this:

    <?php
    function drush_provision_verify() {
      $context = d();
      drush_print('Passed context of type: ' . d()->type);
    }
    ?>

Which would print the type of the context passed to the verify command, in this case 'site'.

The `d()` accessor is not to be feared! It is used all over the place in Aegir and if you're not sure what context you're going to get then you can always print d()->name and you'll get the name of the context that you're dealing with.

### Loading a named context

You can pass an optional argument to the d accessor of the name of the context that you want. So, if you feel compelled to access a property about the main Aegir frontend site anywhere you can do this:

    <?php
      $hostmaster_context = d('@hostmaster');
    ?>

Though most of the time you'll be dealing with the current context, and contexts that it references.

### Subcontexts

Here's an example context:

    <?php
    $aliases['hostmaster'] = array (
      'context_type' => 'site',
      'platform' => '@platform_hostmaster',
      'server' => '@server_master',
      'db_server' => '@server_localhost',
      'uri' => 'aegir.example.com',
      'root' => '/var/aegir/hostmaster-0.4-beta2',
      'site_path' => '/var/aegir/hostmaster-0.4-beta2/sites/aegir.example.com',
      'site_enabled' => true,
      'language' => 'en',
      'client_email' => 'aegir@example.com',
      'aliases' => array (),
      'redirection' => false,
      'profile' => 'hostmaster',
    );

Notice that some of the properties are the name of contexts, such as 'db_server' which has a value of '@server_localhost'.

Suppose now that I have some code that wants to get a property of the db_server associated with the @hostmaster context, I can simply do this:

    <?php
      $db_server_context = d('@hostmaster')->db_server;
      // $db_server_context is now a fully populated context object, not the string
      // '@server_localhost'
      drush_print('DB server on port: . $db_server_context->db_port);
    ?>

The provisionContext object will return full context objects for properties that store the names of other contexts. The how and why of determining which to return as strings and which to return as a context object will be covered later. It is entirely possible to have the name of a context stored as a property in a context, that when accessed returns the string, rather than the context named in the string.


Properties and services
-----------------------

Services are the way in which new properties are added to contexts, in fact if you wish to indicate you want to store additional properties in a context, this must be done by a service. There is no other way to store data in a context.  One might assume that you can just ask provision to save an additional value on a context, but this will not work, and you'll probably get very frustrated indeed!

For example, the `provisionService_pdo` service adds a 'master_db' property to the server context, it does this in a method thus:

    <?php
      function init_server() {
        parent::init_server();
        $this->server->setProperty('master_db');
      }
    ?>

This means that the 'master_db' property can now be set in a `provison-save` Drush command, and will be persisted when the context is saved.

A service may also define properties on the context as actually representing another context, so that you may access that subcontext directly. You can see an example of using this subcontext in the [Implementation section](#implementation) above, but the an example of how you let provision know that a property name is not just a string, but a named context is with a method on the service:

    <?php
      /**
       * Register the http handler for platforms, based on the web_server option.
       */
      static function subscribe_platform($context) {
        $context->setProperty('web_server', '@server_master');
        $context->is_oid('web_server');
        $context->service_subscribe('http',
        $context->web_server->name);
      }
    ?>

Here the `provisionService_http` service is being asked to 'subscribe' to a platform, it sets a property on the context, as above, but additionally uses the `is_oid()` method to indicate that the property is a named context. It then uses that immediately when it gets the name of the web_server context: `$context->web_server->name`. We'll worry about what 'subscribing' to a platform means later.


