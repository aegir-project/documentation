Multiple Web Heads using the Cluster module
===========================================

If you wish to run the same website concurrently on multiple hosts you can use the Pack or Cluster module. 

The **Web Cluster** module uses Rsync to get all files to all servers.

The **Web Pack** module shares files to all servers via an NFS mount.  The NFS setup must be done manually.

This documentation page is about the **Web Cluster** module.

Setting up Web Clusters
-----------------------

In this setup, we will add two web servers and a single database server.

1. Under **Admin > Hosting > Features**, enable the "Web Clusters" module.
2. Setup two or more additional **Remote Servers** using the [Remote Server Instructions](remote-servers.md).
3. Create Remote Server Nodes: Visit **Servers > Add Server** and enter the hostname of your remote server.  For **Web Server**, select *Apache*, *Apache SSL*, *NGINX*, or *NGINX SSL*. 
4. Repeat Step 3 for all of your web servers.
5. Create Cluster Server Node: Visit **Servers > Add Server** and enter any hostname (something like "cluster" so you can identify it.).  For **Web Server**, select *Cluster* and select all of the web servers you wish to add to the cluster. 
6. Setup Database Server: You have two options here. 
  1. Provision a new, separate database server using only the MySQL section from the [Remote Server Instructions](remote-servers.md).  then visit **Servers > Add Server**, enter the new server's hostname, and select "MySQL" for "Database" and enter the root username and password you set during the Remote Server Instructions.
  2. Use an existing server other than "localhost". Aegir installs a database server called "localhost" by default. You cannot use this server for sites installed on remote servesr.  Find the default web server aegir installed (named after the hostmaster hostname).  Click *Edit*, then select "MySQL" for "Database" and enter the root username and password.  If you use the server with the same hostname as the "localhost" database server, you can use the same root username and password.
7. Make sure the Database Server Hostname resolves to the database server IP.  This can be done with DNS or by manually editing the /etc/hosts file on the remote servers.
8. Create or Edit a platform node, and select the cluster server you created in step 5.
9. Create a site node, and select the database server you created in step 6.

If all goes well, the site should install.  

To test that the site is available from both servers, you must edit your DNS or hosts file:

1. Given your site node URL is myclustersite.com, edit /etc/hosts or your DNS so myclustersite.com resolves to the IP of remote server #1.
2. Visit the URL http://myclustersite.com in the browser. If the site works, we know we can load the site from remote server #1.
3. Edit /etc/hosts or your DNS so myclustersite.com resolves to the IP of server #2.
4. Visit the URL http://myclustersite.com again. If the site works, we know we can load the site from remote server #2.

Now you must setup a load balancer that will route requests to both remote server #1 and #2.

Setting up a load balancer is out of scope of this article, for now.
