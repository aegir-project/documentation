Multiple Web Heads using the Pack module
========================================

If you wish to run the same website concurrently on multiple hosts you use the Pack module. Enable the pack module as you would any other drupal module.

A Pack consists of a single server node that is specified as "pack" and any number of server nodes specified as "apache" or "nginx". Aegir rsync's configuration (Apache & Nginx) to "all" nodes in the Pack and rysnc's the platform and site code to only the "master" node in the Pack. All other "slave" nodes will see the platform and site code via NFS.

### Configuring the Pack server node:

The "pack" server node will not be used by Aegir to physically deploy sites or platforms on, so consider it more of a 'virtual server group' for logistical purposes only. When creating the pack node, you do not need to supply a valid Server hostname or IP address, so choose a naming convention that makes sense in your environment.

Next, select the "pack" radio button option under the web configuration when creating or editing the server node, in order to designate this server node as the "pack" server.

Now select the "Master" server from the list of Master servers. The Master server will be the server node that Aegir rsync's the platform and site to. Typically, you would choose the Aegir server itself as the 'master'.

Finally, select the "Slave" servers from the list of Slave servers. The Slave servers will have the Apache or Nginx config rsync'd to them but not the platform or site code, since those will be available to all Slave servers via the NFS share.

### Configuring the Web server nodes (Master and Slaves):

Configure all web server nodes using [these instructions](remote-servers.md). Take care to ensure the 'aegir' user and group on your NFS client machines, have the same UID/GID as that of the NFS server, or else you may run into permissions issues with NFS.

Then mount the files on the remote server.

On the NFS server:

    sudo apt-get install nfs-kernel-server
    echo "/var/aegir/platforms    10.0.0.0/24(rw,no_subtree_check)" >> /etc/exports
    sudo service nfs-kernel-server reload

(Replace the subnet here or add specific /32 hosts as necessary for your environment)

Then on the web servers (Master, if it wasn't also the NFS server, and the Slaves):

    sudo apt-get install nfs-client
    sudo mount 10.0.0.1:/var/aegir/platforms /var/aegir/platforms

(Replace the NFS server's IP here with that of your master server/Aegir server)

Add this to your fstab on the servers that mount the NFS share, so that the share is mounted on boot:

    10.0.0.1:/var/aegir/platforms /var/aegir/platforms nfs rw 0 0

#### Creating a Platform on a Pack:

When configuring a Platform to be deployed on a Pack, choose the "Pack" server node from the Web server radio group during the Platform node creation. Then you choose this Platform as usual when adding a site, and that site will be served from any servers within the Pack.

#### Caveats

Relying on an NFS share to serve your entire /var/aegir/platforms can be a single point of failure if the NFS share becomes unavailable. You may want to look into providing some sort of failover for NFS (google for things like DRBD and Heartbeat), or using some other form of redundancy for your NFS (Netapp filer clusters etc)

Below is an example diagram of a Pack cluster known to be functioning in production (with an optional MySQL-MMM cluster out of scope for this documentation), which may help you visualise the Pack and how it can be put together.

![Diagram of Pack configuration with multiple servers](/img/aegir-pack.png)