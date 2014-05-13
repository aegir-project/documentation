.. index:: Quick Start

Quick Start
-----------

On a recent Debian or Ubuntu server (with a fully qualified domain name), run the following commands:

.. code::

   sudo mysql_secure_installation
   echo "deb http://debian.aegirproject.org stable main" | sudo tee -a /etc/apt/sources.list.d/aegir-stable.list
   curl http://debian.aegirproject.org/key.asc | sudo apt-key add -
   sudo apt-get update
   sudo apt-get install aegir2

See the Installation Guide for more details, or guidance on installing on other operating systems.
