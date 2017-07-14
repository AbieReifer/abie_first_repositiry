## Create Application Containers ##

Application containers are Linux system containers used to host applications and micro-services.
These containers may also host infrastructure components not provided by the Hadoop cluster, such as Redis, or NGINX.
The use of the phrase 'application container' in this context should not be confused with single process container technologies such as Docker.

For example for the NHCS project we have 6 micro-services, which are deployed on one or more of these Linux containers.

As a convention these containers are called ```app999```

### Create Containers ###
Once the baseline node has been configured, copy it as many times as needed.
```shell
# copy the base container to the remote LXD host at hdp1 to create an app container
lxc copy ub1604-base hdp1.westat.com:ub1604-app01-tst
```
#### Update Container specific settings ####
Update the node specific settings, host name and IP address.
Repeat this for each container. This is done on the appropriate host.
```shell
# on the host for this container
lxc file edit ub1604-app01-tst/etc/hostname  # change host name
```
```shell
lxc file edit ub1604-app01-tst/etc/hosts  # change host name and IP address

# ub1604-app01-tst 10.1.245.27
127.0.0.1   localhost
127.0.1.1   hdp-c27.westat.com hdp-c27 # change these

10.1.245.27 hdp-c27.westat.com # change this

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
```shell
lxc file edit ub1604-app01-tst/etc/network/interfaces # change IP address
```

### External Storage ###
Create and mount host ZFS filesystems, as appropriate to the needs of the application.

For example, for NHCS, we create a ZFS file system, and mount it to a location on the container where we will deploy the microservices.
We also create a ZFS file system, and mount it to a container, to store persistent application data.

This configuration happens on the container's host, adjusting the ZFS file system names, container names, and mounted locations as needed.

#### Create ZFS File Systems ####

```shell
# create file systems for each environment, example is for production
sudo zfs create lxd/app
sudo zfs create lxd/app/prd
# APIs are deployed here
sudo zfs create lxd/app/prd/api
# Available to persist data if needed
sudo zfs create lxd/app/prd/data
# Archived data, as needed
sudo zfs create lxd/app/prd/archive
sudo zfs create lxd/app/prd/text-archive

sudo chmod -R 777 /lxd/app
```
#### Mount ZFS File Systems ####
Example is mounting multiple ZFS file systems to the same app container.

```shell
lxc config device add ub1604-app03-prd apidir disk source=/lxd/app/prd/api path=/lxd/api
lxc config device add ub1604-app03-prd archivedir disk source=/lxd/app/prd/archive path=/lxd/archive
lxc config device add ub1604-app03-prd datadir disk source=/lxd/app/prd/data path=/lxd/data

```