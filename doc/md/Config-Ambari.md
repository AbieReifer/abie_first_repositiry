# Install Ambari

The HDP Ambari documentation provides step by step instructions.  

## Set Up Local Repository
As the Ambari and Hadoop containers do not have Internet access, we set up a local Ambari and HDP repository.
See http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/ch_Getting_Ready.html

Setting up the local repository takes place on one of the hosts, we have arbitrarily selected ```hdp0```.

### Obtain the binaries ###
```shell
sudo mkdir /opt/ambari
sudo mkdir /opt/hdp

cd /opt/ambari
sudo wget http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0/ambari.list
sudo wget http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0/ambari-2.4.2.0-ubuntu14.tar.gz
  
cd /opt/hdp
sudo wget http://public-repo-1.hortonworks.com/HDP/ubuntu14/2.x/updates/2.5.3.0/HDP-2.5.3.0-ubuntu14-deb.tar.gz
sudo wget http://public-repo-1.hortonworks.com/HDP/ubuntu14/2.x/updates/2.5.3.0/HDP-2.5.3.0-37.xml
sudo wget http://public-repo-1.hortonworks.com/HDP/ubuntu14/2.x/updates/2.5.3.0/hdp.list
  
sudo wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/ubuntu14/HDP-UTILS-1.1.0.21-ubuntu14.tar.gz
```

### Configure the Repository Server ###
We use NGINX as the application server for the local repository.
```shell
sudo apt install nginx
```
Create the locations which will serve the files.
```shell
sudo mkdir -p /var/www/html/ambari
sudo mkdir -p /var/www/html/hdp
```
Create the server config for the repository.
```shell
# remove the default config
rm /etc/nginx/sites-enabled/*

# create a config file for Ambari
sudo nano /etc/nginx/sites-enabled/ambari

# minimal config for Ambari local repository
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # serve everything under this folder
        root /var/www/html;

        server_name hdp0.westat.com;
        autoindex on;
}
```
Restart nginx for the new configuration to take effect  
```shell
sudo systemctl nginx restart
```
Extract the repository binaries to their repository locations.
```shell
cd /var/www/html/ambari
sudo tar -zxf /opt/ambari/ambari-2.4.2.0-ubuntu14.tar.gz

cd /var/www/html/hdp
sudo tar -zxf /opt/hdp/HDP-2.5.3.0-ubuntu14-deb.tar.gz
sudo tar -zxf /opt/hdp/HDP-UTILS-1.1.0.21-ubuntu14.tar.gz 
```
Make all the files readable by NGINX.
```shell
cd /var/www
sudo chown www-data:www-data -R .
```

### Update Package Manger to Use Local Repository

Modify the ambari.list file.
```shell
sudo nano /opt/ambari/ambari.list

# comment out public entry and point to the new local repository

#VERSION_NUMBER=2.4.2.0-136
#deb http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0 Ambari main

deb http://hdp0.westat.com/ambari/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 Ambari main
```
Push the ambari.list file into the Ambari container.
```shell
lxc file push /opt/ambari/ambari.list ub1404-hdp0-tst/etc/apt/sources.list.d/ambari.list
```

## Set up Java Home
As it will install without Internet access, Ambari will need to know where to find Java.
This takes place in the Ambari container.

```shell
lxc exec ub1404-hdp0-tst -- /bin/bash

# create a profile script for Java
nano /etc/profile.d/java.sh

# set JAVA_HOME
export JAVA_HOME=/opt/jdk/jdk1.8.0_111
export PATH=$PATH:$JAVA_HOME/bin

# make the new profile effective
source /etc/profile
```
## Set up Password-less SSH
This is executed in the Ambari container, such as ub1404-hdp0-prd.
Ambari needs to be able to communicate with all other nodes over SSH, without using a password.
```shell
# generate keys, do not set a password
ssh-keygen -t rsa

# copy key to all nodes, adjust IP addresses as appropriate
ssh-copy-id root@10.1.245.41
ssh-copy-id root@10.1.245.42
ssh-copy-id root@10.1.245.43
```
## Install Ambari
This takes place in the Ambari container.
```shell
sudo apt-get update
sudo apt install ambari-server

# you will receive a warning that packages cannot be authenticated
Warning following packages cannot be authenticated
# proceed anyway
```
## Setup Ambari
This takes place in the Ambari container.
```shell
ambari-server setup

# you will be asked how to install Java
# select Custom JDK and point to location of JAVA_HOME

Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
WARNING: Could not run /usr/sbin/sestatus: OK
Customize user account for ambari-server daemon [y/n] (n)? n
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
[3] Custom JDK
==============================================================================
Enter choice (1): 3
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /opt/jdk/jdk1.8.0_111
Validating JDK on Ambari Server...done.
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? n
Configuring database...
Default properties detected. Using built-in database.
Configuring ambari database...
Checking PostgreSQL...
Configuring local database...
Connecting to local database...done.
Configuring PostgreSQL...
Restarting PostgreSQL
Extracting system views...
.........ambari-admin-2.4.2.0.136.jar
...
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```

## Start Ambari
This takes place in the Ambari container.
```shell
ambari-server start

Using python  /usr/bin/python
Starting ambari-server
Ambari Server running with administrator privileges.
Organizing resource files at /var/lib/ambari-server/resources...
Ambari database consistency check started...
No errors were found.
Ambari database consistency check finished
Server PID at: /var/run/ambari-server/ambari-server.pid
Server out at: /var/log/ambari-server/ambari-server.out
Server log at: /var/log/ambari-server/ambari-server.log
Waiting for server start....................
Ambari Server 'start' completed successfully.
```
Now you are ready to to use Ambari to create and configure the Hadoop cluster.





