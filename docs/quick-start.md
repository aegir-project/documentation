Quick Start Guide
-----------------

On a recent Debian or Ubuntu server (with a fully qualified domain name), run the following commands:
<!-- These quick start instructions are also on https://gitlab.com/aegir/www.aegirproject.org/blob/master/config.yml -->

    $ sudo mysql_secure_installation
    $ echo "deb http://debian.aegirproject.org stable main" | sudo tee -a /etc/apt/sources.list.d/aegir-stable.list
    $ curl http://debian.aegirproject.org/key.asc | sudo apt-key add -
    $ sudo apt-get update
    $ sudo apt-get install aegir3

See the [Installation Guide](/install) for more details, or guidance on installing on other operating systems.
