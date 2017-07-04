### Elasticsearch Base Node Configuration ###

The Elasticsearch base node builds on the ub1604-base node.

Create our Elasticsearch base node from ub1604-base.
```
# on a host
lxc copy ub1604-base ub1604-es0-tst
```

#### OS Configuration ####
Apply same settings for /etc/security/limits.conf and /etc/sysctl.conf as on the host.  
We can do this from the host, using lxc file edit

```shell
lxc file edit ub1604-es0-tst/etc/security/limits.conf

# Recommended for LXD/LXC
*       soft    nofile  1048576
*       hard    nofile  1048576
root    soft    nofile  1048576
root    hard    nofile  1048576
*       soft    memlock unlimited
*       hard    memlock unlimited
```
```shell
lxc file edit ub1604-es0-tst/etc/sysctl.conf

# Recommended for LXD/LXC
fs.inotify.max_queued_events  = 1048576
fs.inotify.max_user_instances = 1048576
fs.inotify.max_user_watches   = 1048576
vm.max_map_count = 262144
```

Check max_map_count, should be 262144
```shell
cat /proc/sys/vm/max_map_count
262144
```

### Install Elasticsearch ###

Elasticsearch containers cannot get Elasticsearch from the Internet.
We need to download the install package and push it into the Elasticsearch base node container.

On a **host** server with Internet access, such as the repository cache host.
```shell
sudo mkdir /opt/elasticsearch
sudo chmod 777 -R /opt/elasticsearch
cd /opt/elasticsearch
sudo wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.1.1.deb
lxc file push /opt/elasticsearch/elasticsearch-5.1.1.deb ub1604-base/opt/
```

The Elasticsearch *.deb package should have been pushed into /opt. 
Start the base node container, connect into it, and install the package.

```shell
lxc start ub1604-es0-tst
lxc exec ub1604-es0-tst -- /bin/bash

sudo dpkg -i /opt/elasticsearch-5.1.1.deb
```

The installation may flag the following warnings such as
```
Couldn't write '65536' to 'vm/mmap_min_addr', ignoring: Permission denied
Couldn't write '1' to 'fs/protected_hardlinks', ignoring: Permission denied
Couldn't write '4 4 1 7' to 'kernel/printk', ignoring: Permission denied
Couldn't write '176' to 'kernel/sysrq', ignoring: Permission denied
Couldn't write '' to 'vm/max_map_count', ignoring: Permission denied
Couldn't write '1' to 'fs/protected_symlinks', ignoring: Permission denied
Couldn't write '1' to 'net/ipv4/tcp_syncookies', ignoring: No such file or directory
Couldn't write '1' to 'kernel/kptr_restrict', ignoring: Permission denied
Couldn't write '1' to 'kernel/yama/ptrace_scope', ignoring: Permission denied
dpkg: error processing package elasticsearch (--install):
```
These are all settings that touch kernel parameters the container cannot change.
The settings have to be made on the physical host. For a standard Ubuntu 16.04 server install
they are likely already set, with the exception of vm/max_map_count, which we inceased previously.  
These warnings can be ignored.

You can check these settings on the host system, as follows.
```
cat /proc/sys/net/ipv4/tcp_syncookies
1
cat /proc/sys/vm/mmap_min_addr
65536
cat /proc/sys/vm/max_map_count
262144
sudo cat /proc/sys/fs/protected_hardlinks
1
sudo cat /proc/sys/fs/protected_symlinks
1
sudo cat /proc/sys/kernel/printk
4 4 1 7
sudo cat /proc/sys/kernel/sysrq
176
sudo cat /proc/sys/kernel/kptr_restrict
1
sudo cat /proc/sys/kernel/yama/ptrace_scope
1
```

#### Elasticsearch Configuration ####

We can perform this configuration outside the container from the host, using lxc file edit ...  
or from inside the container, using sudo nano ...
The example below use the approach from the host.

*JVM Options*    

We assign 6G of RAM to each node.
```shell
lxc file edit ub1604-es0-tst/etc/elasticsearch/jvm.options

# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space
-Xms6g
-Xmx6g
```
*Elasticsearch.yml*  

Set cluster name, node name, paths to data and logs, discovery nodes.
```shell
lxc file edit ub1604-es0-tst/etc/elasticsearch/elasticsearch.yml

cluster.name: nhcs
node.name: ub1604-es0-tst
# node.attr.rack: r1
path.data: /lxd/es/0
path.logs: /lxd/es/0/logs

# bootstrap.memory_lock: true
network.host: 10.1.245.24

# http.port: 9200
discovery.zen.ping.unicast.hosts: ["10.1.245.25", "10.1.245.26"]
discovery.zen.minimum_master_nodes: 2

# gateway.recover_after_nodes: 3
action.destructive_requires_name: true
```
*Memory management*   

Normally we set memory management in the files below:
* /etc/elasticsearch/elasticsearch.yml
* /etc/default/elasticsearch
* /usr/lib/sysctl.d/elasticsearch.conf

However, there appears to be an outstanding issue getting this to work in
LXD containers for ES 5.1.1, so leave ```bootstrap.memory_lock``` commented out in elasticsearch.yml.  

As an option, we set swappiniess to 1. See the recommendations from Elastic, 
https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html#mlockall

To set the swappiness setting, see /usr/lib/sysctl.d/elasticsearch.conf below.

```shell
lxc file edit ub1604-es0-tst/etc/default/elasticsearch

# The maximum number of bytes of memory that may be locked into RAM
# Set to "unlimited" if you use the 'bootstrap.memory_lock: true' option
# in elasticsearch.yml.
# When using Systemd, the LimitMEMLOCK property must be set
# in /usr/lib/systemd/system/elasticsearch.service
MAX_LOCKED_MEMORY=unlimited
```
```shell
lxc file edit ub1604-es0-tst/usr/lib/sysctl.d/elasticsearch.conf

# set swappiness to 1 to minimize memory swapping
VM.SWAPPINESS=1
MAX_OPEN_FILES=66000
MAX_LOCKED_MEMORY=unlimited
MAX_MAP_COUNT=262144
```
```shell
lxc file edit ub1604-es0-tst/usr/lib/systemd/system/elasticsearch.service

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=66000

# Specifies the maximum number of bytes of memory that may be locked into RAM
# Set to "infinity" if you use the 'bootstrap.mlockall: true' option
# in elasticsearch.yml and 'MAX_LOCKED_MEMORY=unlimited' in /etc/default/elasticsearch
LimitMEMLOCK=infinity
```
*Enable Elasticsearch*  
Refresh systemd so that Elasticsearch will start on boot.

```shell
# connect to a terminal in the container
# lxc exec ub1604-es0-tst -- /bin/bash

sudo systemctl enable elasticsearch
sudo systemctl daemon-reload
```

#### Create Elasticsearch Cluster ####
Once the base node is ready, you can create the Elasticsearch cluster.
* [Create Elasticsearch Cluster](./doc/md/CreateCluster-Elasticsearch.md) Create Elasticsearch nodes and set node specific values.

