# Configure Storage 

## Create ZFS Pool and Data Sets
The initial Data Science cluster has been allocated three physical hosts, each of which has four disks.
* Two 300GB SATA disks, configured as a RAID1 by the on-board controller and used for physical host OS
* Two 1.2TB SATA disks configured as a ZFS mirrored pool and used for cluster storage

### Install ZFS
```shell
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt install zfs

# check what disks we have
ls -l /dev/disk/by-path
# check what disks are being used, are mounted
df -h
cat /etc/fstab
```
### Create ZFS Pool
Assume we have two unused disks, /sdb and /sdc, we create a mirrored pool called 'lxd'
```
sudo zpool create lxd mirror sdb sdc
# we may need to force this, in the event the disks were previously used 
sudo zpool create -f lxd mirror sdb sdc

# check pool has been created
sudo zpool list
```
Check ZFS pool is mounted.
See https://www.cyberciti.biz/faq/freebsd-linux-unix-zfs-automatic-mount-points-command/
```shell
sudo zfs get mounted lxd
# if not mounted
sudo zfs set mountpoint=/lxd lxd
```
## Create ZFS file systems for cluster nodes

In this initial version of the Data Science cluster, each physical host has two physical drives allocated for data storage.
The two drives are configured as a ZFS mirror, called lxd, and made available to the OS as /lxd

Everything running on the cluster has to share these drives. This incudes 3 complete environments for the NHCS project.

To provide for those three environments, and to mimic a cluster where each data node would have its own physical storage, ZFS file systems
are created for each clustered application, for each node, in each environment. It is critical each HDP data node have its own file system to
avoid errors which would occur if two different data nodes were trying to use the same underlying file sytem location.

The naming convention on the physical host is:
``` /lxd/[application]/[environment]/[node] ```  

For example:
``` /lxd/hdp/prd/0 ``` is for HDP in the production environment, node 0.

These ZFS file systems have to be created on the physical host, then configured
as an external data store in each container.

For example, to create the ZFS file systems on hdp0 which is hosting node 0 and 3 for HDP, and node 0 for Elasticsearch:
```
# HDP Test
sudo zfs create lxd\hdp
sudo zfs create lxd\hdp\tst
sudo zfs create lxd\hdp\tst\0
sudo zfs create lxd\hdp\tst\3
# HDP Staging
sudo zfs create lxd\hdp\stg
sudo zfs create lxd\hdp\stg\0
sudo zfs create lxd\hdp\stg\3
# HDP Production
sudo zfs create lxd\hdp\prd
sudo zfs create lxd\hdp\prd\0
sudo zfs create lxd\hdp\prd\3

# Elasticsearch Test
sudo zfs create lxd\es
sudo zfs create lxd\es\tst
sudo zfs create lxd\es\tst\0
# Elasticsearch Staging
sudo zfs create lxd\es\stg
sudo zfs create lxd\es\stg\0
# Elasticsearch Production
sudo zfs create lxd\es\prd
sudo zfs create lxd\es\prd\0
```

Some physical hosts support Kafka, in this example we create a ZFS file system for Kafka in Production
```
sudo zfs create lxd\kafka
sudo zfs create lxd\kafka\prd
sudo zfs create lxd\kafka\prd\0
```

Some physical hosts support Redis nodes, in this example we create a ZFS file system for Redis in Production
```
sudo zfs create lxd\redis
sudo zfs create lxd\redis\prd
sudo zfs create lxd\redis\prd\0
```
After creating all the ZFS file systems, you should have something like the following.
```
# list all filesystems for an application, in this case on hdp0 for HDP, we see file systems for all environments
sudo zfs list | grep hdp
lxd/hdp                                                                               152K  1.01T    19K  /lxd/hdp
lxd/hdp/prd                                                                            57K  1.01T    19K  /lxd/hdp/prd
lxd/hdp/prd/0                                                                          19K  1.01T    19K  /lxd/hdp/prd/0
lxd/hdp/prd/3                                                                          19K  1.01T    19K  /lxd/hdp/prd/3
lxd/hdp/stg                                                                            57K  1.01T    19K  /lxd/hdp/stg
lxd/hdp/stg/0                                                                          19K  1.01T    19K  /lxd/hdp/stg/0
lxd/hdp/stg/3                                                                          19K  1.01T    19K  /lxd/hdp/stg/3
lxd/hdp/tst                                                                            19K  1.01T    19K  /lxd/hdp/tst
lxd/hdp/tst/0                                                                          19K  1.01T    19K  /lxd/hdp/tst/0
lxd/hdp/tst/0                                                                          19K  1.01T    19K  /lxd/hdp/tst/3

# list all filesystems for an environment, in this case on hdp0 for production, we see file systems for all applications
sudo zfs list | grep prd
lxd/es/prd                                                                           6.48G  1.01T    19K  /lxd/es/prd
lxd/es/prd/0                                                                         6.48G  1.01T  6.48G  /lxd/es/prd/0
lxd/hdp/prd                                                                            57K  1.01T    19K  /lxd/hdp/prd
lxd/hdp/prd/0                                                                          19K  1.01T    19K  /lxd/hdp/prd/0
lxd/hdp/prd/3                                                                          19K  1.01T    19K  /lxd/hdp/prd/3
lxd/redis/prd                                                                          19K  1.01T    19K  /lxd/redis/prd
```

## Set Permissions on external storage

The LXD containers are running in unprivileged mode, that is the user and group Ids have been shifted such that 
a user in the container has no privileges on the host.

To allow the root user in the container to write to the external storage locations that have been mounted in the container 
we need to set permissions on those locations.

The root user in the LXD container defaults to  ```uid:gid``` of ```100000:100000```.  So we set ownership on the external storage locations 
appropriately. This is done on the physical host.
```shell
# example is for HDP and Elasticsearch
sudo chown 100000:100000 -R /lxd/hdp
sudo chown 100000:100000 -R /lxd/es
```
**Note: if permissions are not set correctly, Ambari will fail trying to install Hadoop services which includes changing the ownership of some directories**










