Hosting - the Aegir frontend
=============================

[TOC]

The Aegir frontend is a Drupal site in itself. You can use any Drupal module to extend it.



Adding a module
-------------

Add a new module either to the site specific moduled directory e.g. `/var/aegir/hostmaster-7.x-3.x/sites/aegir.example.com/modules` or use [your own makefile](http://docs.aegirproject.org/en/3.x/install/#71-custom-make-files) to create the hostmaster platform.


UI modules
----------

To change one of the content types of views used in the Aegir frontend make sure you have the views_ui and field_ui core modules enabled.
You can enable the like any other Drupal module as they are included with code already.


Interacting via code
--------------------

You can also use your own code to interacti with the frontend.

To e.g. create a task you could run this little script from inside the `/var/aegir/hostmaster-7.x-3.x/sites/example.com/` directory.

```php
#!/usr/bin/env drush
<?php

$node = new stdClass();

// Admin
$node->uid = 1;

$node->type = 'task';

// The site, platform or server node ID that is subject to the task.
// 10 usually is the node ID for the hostmaster site itself.
$node->rid = 10;

// Published status == 1
$node->status = 1;

$node->task_type = 'verify';

// Setting status to HOSTING_TASK_QUEUED == 0
$node->task_status = 0;

node_save($node);

echo "The full node as it was created: ";
print_r($node);
```

Note that this adds a task to the queue, it's up to the queue daemon or a cronjob to start it.

