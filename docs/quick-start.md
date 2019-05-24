Quick Start Guide
-----------------

### For servers with fully-qualified domain names

On a recent Debian or Ubuntu server with a fully-qualified domain name (FQDN), run the following commands:
<!-- These quick start instructions are also on https://gitlab.com/aegir/www.aegirproject.org/blob/master/config.yml -->

```sh
sudo mysql_secure_installation
echo "deb http://debian.aegirproject.org stable main" | sudo tee -a /etc/apt/sources.list.d/aegir-stable.list
curl http://debian.aegirproject.org/key.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install aegir3
```

### For servers without fully-qualified domain names

To get a Aegir instance for trial purposes on a temporary VM (either local or remote) without an FQDN, follow the [Aegir Development VM](https://gitlab.com/aegir/aegir-dev-vm) instructions.

### For more information

See the [Installation Guide](/install) for more details, or guidance on installing on other operating systems.
