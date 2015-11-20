Quick Start Guide
-----------------

On a recent Debian or Ubuntu server (with a fully qualified domain name), run the following commands:

    $ sudo mysql_secure_installation
    $ echo "deb http://debian.aegirproject.org unstable main" | sudo tee -a /etc/apt/sources.list.d/aegir-unstable.list
    $ curl http://debian.aegirproject.org/key.asc | sudo apt-key add -
    $ sudo apt-get update
    $ sudo apt-get install aegir3

See the [Installation Guide](/install) for more details, or guidance on installing on other operating systems.
