# Prepare Hadoop Nodes

* Copy the Hadoop baseline container to create the desired number of nodes
* Update the node specific configuration for each container (hostname and IP address)
* Mount external storage to each container

## Create Nodes
Once the baseline node has been configured, copy it as many times as needed.
```shell
# assume working on hdp0 host
lxc copy ub1404-base ub1404-hdp0-prd
lxc copy ub1404-base ub1404-hdp3-prd
lxc copy ub1404-base hdp1.westat.com:ub1404-hdp1-prd # copy container to hdp1 host
lxc copy ub1404-base hdp2.westat.com:ub1404-hdp2-prd # copy container to hdp2 host
```
## Update Container specific settings
Update the node specific settings, host name and IP address.
Repeat this for each container/node. Example is for c43, these command are
executed from the physical host.

Set host name
```shell
lxc file edit ub1404-hdp3-prd/etc/hostname
# update fully qualified hosst name
hdp-c43.westat.com
```
Verify hosts, this should not need to be changed
```shell 
lxc file edit ub1404-hdp3-prd/etc/hosts 

127.0.0.1   localhost
# do not assign host name to localhost IP, this messes up Zookeeper
# 127.0.1.1   hdp-c43.westat.com hdp-c43

# add all cluster servers
10.1.245.40 hdp-c40.westat.com
10.1.245.41 hdp-c41.westat.com
10.1.245.42 hdp-c42.westat.com
10.1.245.43 hdp-c43.westat.com
```
Update the IP address
```shell
lxc file edit ub1404-hdp3-prd/etc/network/interfaces
# change IP address
auto eth0
iface eth0 inet static
   address 10.1.245.43
   gateway 10.1.245.1
   netmask 255.255.255.0
   network 10.1.245.0
   broadcast 10.1.245.255
   dns-nameservers 10.13.9.225 10.13.9.226
```
After hostname and network settings have been updated, stop and start the container.

## Mount External Storage

Previously we created ZFS file systems on the physical host to serve as external storage for each container. 
Now we are going to mount those external storage locations to the containers.

The assumptions in the naming convention shown are:
* a container would never particiapte in two environments, such a simultaneously in tst and prod
* a container may host more than one application, such as HDP and Elasticsearch, 
but would not host more than one node within a clustered application. That is
a container may have /lxd/hdp and /lxd/es file systems, but would never have /lxd/hdp/0 and /lxd/hdp/1.

The naming convention for the LXD storage device name is ```[application name]dir``` such as ```hdpdir```.

Examples;  
Configure external storage for containers hosting HDP nodes in the production environment:
```shell
## On hdp0 which is physical host for nodes 0 and 3
lxc config device add ub1404-hdp0-prd hdpdir disk source=/lxd/hdp/prd/0 path=/lxd/hdp
lxc config device add ub1404-hdp3-prd hdpdir disk source=/lxd/hdp/prd/3 path=/lxd/hdp

## On hdp1 which is physical host for node 1
lxc config device add ub1404-hdp1-prd hdpdir disk source=/lxd/hdp/prd/1 path=/lxd/hdp

## On hdp2 which is physical host for node 2
lxc config device add ub1404-hdp2-prd hdpdir disk source=/lxd/hdp/prd/2 path=/lxd/hdp

```
Configure external storage for containers hosting Elasticsearch nodes in the production environment:
```shell
## On hdp0 which is physical host for node 0 
lxc config device add ub1404-es0-prd esdir disk source=/lxd/es/prd/0 path=/lxd/es

## On hdp1 which is physical host for node 1 
lxc config device add ub1404-es1-prd esdir disk source=/lxd/es/prd/1 path=/lxd/es

## On hdp2 which is physical host for node 2
lxc config device add ub1404-es2-prd esdir disk source=/lxd/es/prd/2 path=/lxd/es
```
Configure external storage for container(s) hosting Redis cache in the production environment:
```shell
# in this example Redis is being deployed in the same container as HDP node 2, but given its own ZFS file system
lxc config device add ub1404-hdp2-prd redisdir disk source=/lxd/redis/prd/0 path=/lxd/redis
```
### Check External Storage

Check external storage hasa been mounted, example is for ub1404-hdp3-stg
```shell
lxc config device show ub1404-hdp3-stg
br0:
  nictype: bridged
  parent: br0
  type: nic
root:
  path: /
  type: disk
hdpdir:
  path: /lxd/hdp
  source: /lxd/hdp/stg/3
  type: disk
```

