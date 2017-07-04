### Hadoop Baseline Node Configuration ###

The Hadoop base node includes configuration specific to a Hadoop node.
This configuration prepares the container to be part of a Hadoop cluster, using Ambari as
the cluster installer and manager.

#### Create Ambari Node ####
Ambari will be installed on its own node, in Test this is ub1404-hdp0-tst.

Create the Ambari node for the Test environment
```shell
# on a host
lxc copy ub1404-base ub1404-hdp0-tst
```
#### Configure Host name and Hosts  ####
Set fully qualified hostname
```shell
lxc file edit ub1404-hdp0-tst/etc/hostname
# fully qualified name of this node
hdp-c20.westat.com
````
Set IP addresses of all other nodes in the cluster
```shell
lxc file edit ub1404-hdp0-tst/etc/hosts
127.0.0.1   localhost localhost.localdomain

# really important we do NOT add an entry here like
# 127.0.0.1 hdp-c20.westat.com
# If we do, Zookeeper listens on wrong interface

# add all nodes in the cluster
10.1.245.20 hdp-c20.westat.com
10.1.245.21 hdp-c21.westat.com
10.1.245.22 hdp-c22.westat.com
10.1.245.23 hdp-c23.westat.com

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
#### Create a Hadoop base node ####
We copy our ub1404-hdp0 node to create the Hadoop base node.
```shell
# on the host for ub1404-hdp0-tst, we are copying the container to a remote LXD on hdp1
lxc copy ub1404-hdp0-tst hdp1.westat.com:ub1404-hdp1-tst

# edit the new container and update network settings
# this happens on the remote LXD, hdp1
lxc file edit ub1404-hdp1-tst/etc/hostname # change host name
lxc file edit ub1404-hdp1-tst/etc/network/interfaces # change IP address
```
#### No Password SSH ####
To allow Ambari to manage nodes, we need to allow root to login over SSH.
Do this on our Hadoop base node, ub1404-hdp1
```shell
lxc start ub1404-hdp1-tst
lxc exec ub1404-hdp1-tst -- /bin/bash

# change SSH config to allow root login over SSH
nano /etc/ssh/sshd_config

# Authentication:
LoginGraceTime 120
# PermitRootLogin without-password
PermitRootLogin yes
StrictModes yes

# restart ssh
sudo /etc/init.d/ssh restart

# assign root a password
sudo passwd root
```
Later, from the Ambari node, we will copy the private key from the Ambari node to each node in the cluster.
