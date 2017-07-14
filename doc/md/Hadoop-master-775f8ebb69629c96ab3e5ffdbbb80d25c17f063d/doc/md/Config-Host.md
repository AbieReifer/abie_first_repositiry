# Host Configuration

Each physical host is set up to support both Hadoop nodes and Elasticsearch nodes running in Linux containers.

**One** of the hosts will act as a repository cache for the entire cluster.  
**One** of the hosts may also act as NTP server for the entire cluster.  

## Set up a Repository Cache
Use apt-cacher-ng to set up a host as a repository cache.  This only occurs on *ONE* host.
All servers (hosts and containers) will be configured to use the cache as proxy when ```apt``` is invoked.

We, arbitrarily, chose **hdp1** as the repository cache server.

See https://www.unix-ag.uni-kl.de/~bloch/acng/

Install Apt-Cacher-NG.
```shell
# We are doing this on hdp1
sudo apt-get install apt-cacher-ng
```
Update the configuration file. The only required change is the addition of a line to support updating
Oracle Java, which requires accepting a license agreement. Only those settings which need to be verified are
included in the portion of the file below.
```shell
sudo nano /etc/apt-cacher-ng/acng.conf

# PfilePatternEx:
# The following two lines are to support updating Oracle Java through apt-cacher-ng
# see http://seotoast.com/install-oracle-java7-installer-through-apt-cacher-ng/
# see https://gist.github.com/makuk66/5639157
 
PfilePattern = .*(\.deb|\.rpm|\.dsc|\.tar\.gz\.gpg|\.tar\.gz|\.tgz|\.zip|\.diff\.gz|\.diff\.bz2|\.jigdo|\.bin(\?AuthParam=.*)?|\.template|changelog|copyright|\.udeb|\.diff/.*\.gz|vmlinuz|initrd\.gz|(Devel)?ReleaseAnnouncement(\\?.*)?)$
RequestAppendix: Cookie: oraclelicense=a

# VfilePatternEx:
# SPfilePatternEx:
# SVfilePatternEx:
# WfilePatternEx:
```
Check the server is running by getting statistics.  
http://10.1.245.11:3142/acng-report.html 

## Use Repository Cache (apt-cacher-ng)
Now that we have a repository cache, let's use it. Configure this machine to use the repository cache server.
We have designated hdp1 as the cache server, thus the proxy is http:10.1.245.11:3142
```shell
sudo nano /etc/apt/apt.conf.d/02proxy
# point proxy to the repository server for the cluster
Acquire::http { Proxy "http:10.1.245.11:3142"; };
```

## Install Core Linux Utilities
Some of these may already be installed.
```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt install nano ssh wget ntp ntpstat curl zip unzip dnsutils nscd bridge-utils
```

## Networking
The host's two physical NICS are bonded to provide fail over, then assigned to a bridge ```br0``` used by the
Linux containers.

Ensure bridge-utils is installed.
```shell
sudo apt install bridge-utils
```
Edit network configuration (adjust interface names)
```shell
sudo nano /etc/network/interfaces 

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# slave interface
auto eno1
iface eno1 inet manual
   bond-master bond0

# slave interface
auto eno2
iface eno2 inet manual
   bond-master bond0

# bonded interface, slaves eno1 eno2
auto bond0
iface bond0 inet manual
   bond-mode 802.3ad
   bond-miimon 100
   bond-lacp-rate 1
   bond-slaves eno1 eno2

# bridge on bonded interface, will NAT all traffic to containers
auto br0
iface br0 inet static
   address 10.1.245.10
   gateway 10.1.245.1
   network 10.1.245.0
   broadcast 10.1.245.255
   netmask 255.255.255.0
   dns-nameservers 10.13.9.225 10.13.9.226
   bridge_ports bond0
   bridge_stp off
   bridge_fd 0
   bridge_hello 2
   bridge_maxage 12

# restart networking
sudo systemctl restart networking

# check network connectivity
ping google.com
```
## Install LXD /LXC
LXD is part of the base Ubuntu server install. However, we want the latest version, so we use the ppa: LXD repository.
```shell
sudo add-apt-repository ppa:ubuntu-lxc/lxd-stable
sudo apt-get update
# we do a dist-upgrade to get the latest version of LXD
sudo apt-get dist-upgrade
```
Run the LXD initialization. Answer prompts as shown.
```shell
sudo lxd init

Name of the storage backend to use (dir or zfs) [default=zfs]:
Create a new ZFS pool (yes/no) [default=yes]? no
Name of the existing ZFS pool or dataset: lxd
Would you like LXD to be available over the network (yes/no) [default=no]? yes
Address to bind LXD to (not including port) [default=all]:
Port to bind LXD to [default=8443]:
Trust password for new clients:
Again:
Do you want to configure the LXD bridge (yes/no) [default=yes]? no
```
Configure default LXD profile to use the ```br0``` interface created earlier.
```shell
lxc profile edit default

name: default
config: {}
description: "Default LXD profile"
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: br0
    type: nic
```
Attach our br0 network to the default profile. All containers created after this will use the
default profile and network through br0.
```shell
lxc network attach-profile br0 default eth0
```

## OS Settings

### LXD/LXC Requirements

*Configure max_map_count* 
```shell
sudo nano /proc/sys/vm/max_map_count
262144
```
*Increase limits and maximum file handles*  
See https://github.com/lxc/lxd/blob/master/doc/production-setup.md 
Add these settings to the end of the following files.

```shell
sudo nano /etc/security/limits.conf

# recommended for linux containers
*       soft   nofile	1048576
*	    hard   nofile	1048576
root	soft   nofile	1048576
root	hard   nofile	1048576
*	    soft   memlock	unlimited
*	    hard   memlock	unlimited
```
```shell
sudo nano /etc/sysctl.conf

# recommended for linux containers
fs.inotify.max_queued_events  = 1048576
fs.inotify.max_user_instances = 1048576
fs.inotify.max_user_watches   = 1048576
vm.max_map_count = 262144
```

### Hadoop Requirements

*Create Hadoop storage locations*  
Create ZFS file systems in the pool, one for each Hadoop node/container on the host, where 0 is the node number. Later we will mount these
to each Hadoop container.
```shell
sudo zfs create lxd/hdp
sudo zfs create lxd/hdp/0
sudo zfs create lxd/hdp/1
sudo zfs create lxd/hdp/2
sudo zfs create lxd/hdp/3
```
For now, make these ZFS file systems world writeable.
```TO DO``` set up UID and GID to designate specific user in container which can read/write to host.
```
sudo chmod 777 -R /lxd/hdp
```

*Disable Transparent HugePages on Host*   
The default setting on Ubuntu for transparent_hugepage is [always]. This has to be changed to [never],
if this is not disabled, Ambari agents running in containers on the host fail to register.

Check status
```shell
cat /sys/kernel/mm/transparent_hugepage/enabled
```
If status is [always] disable this at boot using a command in GRUB, as follows. Note the GRUB_CMDLINE_LINUX_DEFAULT entry.
```shell
sudo nano /etc/default/grub

# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0
GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="transparent_hugepage=never"
GRUB_CMDLINE_LINUX=""

# after making the change, update GRUB before a reboot
sudo update-grub
```
See http://www.bictor.com/2015/07/19/disabling-transparent-hugepages-in-ubuntu-14-04-server-lts/ 

Reboot the server after making this change.

### Elasticsearch Requirements

*Create Elasticsearch storage locations*   
Create ZFS file systems in the pool, one for each Elasticsearch node/container on the host, where 0 is the node number. Later we will mount these
to each Elasticsearch container.
```shell
sudo zfs create lxd/es
sudo zfs create lxd/es/0
sudo zfs create lxd/es/1
sudo zfs create lxd/es/2
```
For now, make these ZFS file systems world writeable.
```TO DO``` set up UID and GID to designate specific user in container which can read/write to host.
```
sudo chmod 777 -R /lxd/es
```

## Set up LXD Remotes

Once all the hosts have been configured, they can be set up as remotes for each other.  
See https://www.stgraber.org/2016/04/12/lxd-2-0-remote-hosts-and-container-migration-612

Adjust commands below for the DNS names and IP addresses of the other hosts.
```shell
lxc remote add hdp0.westat.com 10.1.245.10
lxc remote add hdp1.westat.com 10.1.245.11
```
Copy baseline nodes to remaining hosts.
```shell
# assume base nodes were set up on hdp0
lxc stop ub1404-base
lxc stop ub1604-base
lxc copy ub1404-base hdp1.westat.com:ub1404-base
lxc copy ub1404-base hdp2.westat.com:ub1404-base

lxc copy ub1604-base hdp1.westat.com:ub1604-base
lxc copy ub1604-base hdp2.westat.com:ub1604-base
```

## Create NTP Server (Deprecated)   
**Deprecated** we now use ```ntp2.westat.com``` as NTP server.

Clustered nodes are very sensitive to time differences.
We set up a NTP server to broadcast to the cluster.

This is configured on one of the hosts.

```shell
sudo apt-get install ntp ntp-doc

# /etc/ntp.conf, configuration for ntpd; see ntp.conf(5) for help
driftfile /var/lib/ntp/ntp.drift
# Specify one or more NTP servers.
server 0.north-america.pool.ntp.org iburst
server 1.north-america.pool.ntp.org
server 2.north-america.pool.ntp.org
server 3.north-america.pool.ntp.org

# Use Ubuntu's ntp server as a fallback.
server ntp.ubuntu.com
server 127.127.1.0
fudge 127.127.1.0 stratum 10

# By default, exchange time with everybody, but don't allow configuration.
restrict -4 default kod notrap nomodify nopeer noquery
restrict -6 default kod notrap nomodify nopeer noquery

# Local users may interrogate the ntp server more closely.
restrict 127.0.0.1
restrict ::1

# If you want to provide time to your local subnet, change the next line.
# (Again, the address is an example only.)
broadcast 10.1.245.255
```

The configuration of the baseline nodes follows, it is identical for both baselines.  
See [Configure Baseline Node](./doc/md/Config-BaseNode.md)

