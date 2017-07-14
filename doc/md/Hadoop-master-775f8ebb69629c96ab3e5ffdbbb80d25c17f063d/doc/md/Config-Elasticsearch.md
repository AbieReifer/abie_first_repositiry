## Create Elasticsearch Cluster ##

* Copy Elasticsearch baseline node to create desired number of nodes
* Update node specific configuration (hostname, IP address, elasticsearch.yml)
* Mount external storage to each container

All these actions take place on the hosts on which the containers will run.

#### Create Nodes ####
Once the Elasticsearch baseline node has been configured, copy it as many times as needed.
```shell
# copy Elasticsearch base node to remote LXD on hdp1
lxc copy ub1604-es0-tst hdp1.westat.com:ub1604-es1-tst

# copy Elasticsearch base node to remote LXD on hdp2
lxc copy ub1604-es0-tst hdp2.westat.com:ub1604-es2-tst
```
#### Update Container specific settings ####
Update the node specific settings, host name and IP address.
Repeat this for each container. This is done on the appropriate host.

```shell
lxc file edit ub1604-es2-prd/etc/security/limits.conf
lxc file edit ub1604-es2-prd/etc/ntp.conf

lxc file edit ub1604-es2-prd/etc/hostname
lxc file edit ub1604-es2-prd/etc/hosts
lxc file edit ub1604-es2-prd/etc/network/interfaces

lxc file edit ub1604-es2-prd/etc/elasticsearch/jvm.options
lxc file edit ub1604-es2-prd/etc/elasticsearch/elasticsearch.yml

sudo zfs create lxd/es/prd
sudo zfs create lxd/es/prd/0
sudo chmod 777 -R /lxd/es/prd

lxc config device remove ub1604-es2-prd zfsdir
lxc config device add ub1604-es2-prd zfsdir disk source=/lxd/es/prd/0 path=/lxd/es/0

lxc start ub1604-es2-prd
lxc exec ub1604-es2-prd -- /bin/bash
nano /etc/profile.d/java.sh
mkdir /lxd/es/0/logs

sudo systemctl stop elasticsearch
sudo systemctl start elasticsearch
tail -n100 /lxd/es/0/logs/nhcsprd.log
```

```shell
# on the host for this container
lxc file edit ub1604-es1-tst/etc/hostname  # change host name
```
```shell
lxc file edit ub1604-es1-tst/etc/hosts  # change host name and IP address

# ub1604-es2-tst
127.0.0.1   localhost
127.0.1.1   hdp-c25.westat.com hdp-c25 # change these

10.1.245.25 hdp-c25.westat.com # change this

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
```shell
lxc file edit ub1604-es1-tst/etc/network/interfaces # change IP address

# change values for all of:  node.name, path.data, path.logs, network.host, discovery.zen.ping.unicast.hosts
lxc file edit ub1604-es1-tst/etc/elasticsearch/elasticsearch.yml 
```
#### Mount External Storage ####
Mount the host ZFS filesystem specific to each container.
This has to happen on the container's host, adjusting the container names and data locations as needed.
```shell
lxc config device add ub1604-es0-tst zfsdir disk source=/lxd/es/0 path=/lxd/es/0

lxc config device add ub1604-es1-tst zfsdir disk source=/lxd/es/1 path=/lxd/es/1
lxc config device add ub1604-es2-tst zfsdir disk source=/lxd/es/2 path=/lxd/es/2
```

