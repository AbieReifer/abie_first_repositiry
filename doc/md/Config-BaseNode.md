# Create Baseline Nodes

As a practical matter, the approach to configuring nodes is to create and configure a "baseline" container, 
then copy this container to create as many nodes as desired, updating them as needed.

The common baseline containers, ub1404-base and ub1604-base, have an identical configuration which includes:
* Use apt-cacher-ng to pull libraries from the cluster's repository cache server
* Linux utilities, nano ssh wget ntp curl unzip dnsutils nscd bridge-utils
* Oracle Java 1.8
* NTP client, using a name server created/defined for the cluster
* Hostname and hosts settings
* Network configuration with a static IP address
* Zabbix agent for monitoring

From the common baseline nodes, two more specific baseline nodes are created:  
* [Hadoop Base Node](./doc/md/Config-BaseNode-Hadoop.md) built on ub1404-base, with Hadoop specific configuration.
* [Elasticsearch Base Node](./doc/md/Config-BaseNode-ES.md) built on ub1604-base, with an Elasticsearch installation and configuration.

## Create LXD Base Nodes ##

The baseline nodes only have to be created **ONCE** on one of the hosts.

We create two baseline nodes;
* ub1404-base: used for Hadoop nodes, this is the latest version of Ubuntu supported by the Horton DataWorks Platform
* ub1604-base: used for all other nodes, including Elasticsearch

Show available images, and create base nodes.
```shell
# show available images, the images: repository is configured by default
lxc image list images:
# create base nodes from a Ubuntu 14.04 image and a Ubuntu 16.04 image
lxc launch images:ubuntu/trusty ub1404-base
lxc launch images:ubuntu/xenial ub1604-base

# both containers should now be running, check this
lxc list
```
The following configuration steps are applied to both ub1404-base and ub1604-base.

### Configure Network

**Set fully qualified hostname**
```shell
lxc file edit ub1404-base/etc/hostname
# fully qualified name of this node
hdp-c20.westat.com
```
**Set hosts**
```shell
lxc file edit ub1404-base/etc/hosts
127.0.0.1   localhost localhost.localdomain
```
**Set fixed IP address**
```shell
# for ub1404-base
lxc file edit ub1404-base/etc/network/interfaces
# for ub1604-base
lxc file edit ub1604-base/etc/network/interfaces

# make sure iface starts on a new line
auto eth0  
iface eth0 inet static   
   address 10.1.245.20 
   gateway 10.1.245.1  
   network 10.1.245.0  
   broadcast 10.1.245.255 
   netmask 255.255.255.0    
   dns-nameservers 10.13.9.225 10.13.9.226  
```
### Install core libraries
For the remainder of the node configuration, we need to be working on the node.
```shell
lxc list
# if the container is running, stop it
lxc stop ub1404-base
# start container to pick up network config
lxc start ub1404-base  
# enter the container
lxc exec ub1404-base -- /bin/bash

# check network is correctly set up
cat /etc/hostname
cat /etc/hosts
ifconfig
# check local network copnnectivity
ping hdp1.westat.com
```
### Install Oracle Java SDK

Firewall rules prevent containers from accessing the Internet.  
As a result we cannot get and update Java using a ppa.

To provide Java to the containers, we copy the Java SDK to a location on the 
cache server, then push the Java SDK into the base nodes.

On a **host** server, such as the repository cache host.
```shell
# this occurs on a host node with Internet access
sudo mkdir /opt/java
sudo chmod 777 -R /opt/java
cd /opt/java
sudo wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u111-b14/jdk-8u111-linux-x64.tar.gz

# push the SDK tar file to the base nodes
lxc file push /opt/java/jdk-8u111-linux-x64.tar.gz ub1404-base/opt/
lxc file push /opt/java/jdk-8u111-linux-x64.tar.gz ub1604-base/opt/
```
The Java SDK should have been pushed into the container under /opt
```shell
# this occurs in the container
sudo mkdir /opt/jdk
cd /opt
# untar the SDK into /opt/jdk
sudo tar -zxf jdk-8u111-linux-x64.tar.gz -C /opt/jdk
# tell the OS Java is at that location
sudo update-alternatives --install /usr/bin/java java /opt/jdk/jdk1.8.0_111/bin/java 100
sudo update-alternatives --install /usr/bin/javac javac /opt/jdk/jdk1.8.0_111/bin/javac 100

# verify java is available
java -version
javac -version
```

### Set up Java Home
On all containers using Java (pretty much everything) set up the JAVA_HOME envirionment.

```shell
# this occurs on a base container
# create a profile script for Java
nano /etc/profile.d/java.sh

# set JAVA_HOME
export JAVA_HOME=/opt/jdk/jdk1.8.0_111
export PATH=$PATH:$JAVA_HOME/bin

# make the new profile effective
source /etc/profile
```

### Configure Repository Cache (apt-cacher-ng)
```shell
vim /etc/apt/apt.conf.d/02proxy
# point proxy to the repository server for the cluster
Acquire::http { Proxy "http:10.1.245.11:3142"; };
```
### Install Linux Utilities
```shell
sudo apt-get install nano ssh wget ntp curl unzip dnsutils nscd bridge-utils
```
### NTP Client
Configure node to be a client for a cluster NTP server.
NTP server designated as ```ntp2.westat.com```, change as needed

```shell
sudo nano /etc/ntp.conf
# /etc/ntp.conf, configuration for ntpd; see ntp.conf(5) for help

driftfile /var/lib/ntp/ntp.drift

# Enable this if you want statistics to be logged.
#statsdir /var/log/ntpstats/

statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable

# Specify one or more NTP servers.

server ntp2.westat.com

# Note that "restrict" applies to both servers and clients, so a configuration
# that might be intended to block requests from certain clients could also end
# up blocking replies from your own upstream servers.

# By default, exchange time with everybody, but don't allow configuration.
restrict -4 default kod notrap nomodify nopeer noquery
restrict -6 default kod notrap nomodify nopeer noquery

# Local users may interrogate the ntp server more closely.
restrict 127.0.0.1
restrict ::1

# If you want to listen to time broadcasts on your local subnet, de-comment the
# next lines.  Please do this only if you trust everybody on the network!

disable auth
broadcastclient

# verify NTP servers
ntpq -p
```

### Install Zabbix Agent
See [Configure Zabbix Agent](./doc/md/Config-Zappix-Agent.md) 
