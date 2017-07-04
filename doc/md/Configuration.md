# Configuration #

All nodes in both the Hadoop and Elasticsearch clusters run in Linux system containers (LXC, see https://linuxcontainers.org/lxd/) 
managed using the LXD daemon. We have adopted a structure where there is a one-to-one relationship between node and container; that is each 
container hosts a single node, either in the Hadoop or Elasticsearch cluster. Containers could, of course, host nodes from multiple clusters, but
using LXC containers there is very little efficiency to gain from that approach, and it adds significant complexity and possible sources of resource contention.
As a result node and container can be considered interchageable terms in this documentation.

The only exception to this is Redis, which, simply for lack of container IP addresses, is hosted in a container which also hosts a Hadoop node. Ideally, if more IP
addreses were available, Redis would have its own container(s).

All cluster storage uses ZFS as the local filesytem for each node/container, see overview at https://en.wikipedia.org/wiki/ZFS and Linux version at https://github.com/zfsonlinux/zfs

Configuration involves setting up the physical hosts which will host the Linux containers, configuring LXD, configuring ZFS pools and file systems to provide storage to cluster nodes,
configuring the cluster nodes/containers, and finally installing the cluster application in the containers.

## Infrastructure
* [Configure Hosts](./doc/md/Config-Host.md) : Setting up the physical hosts in the cluster.
* [Configure Storage](./doc/md/Config-Storage.md) : Configure and allocate storage to nodes / containers.
* [Create Baseline Node](./doc/md/Config-BaseNode.md) : Creating and configuring a baseline node.

## Hadoop
* [Create Baseline Hadoop Node](./doc/md/Config-BaseNode-Hadoop.md) : Extend the baseline node to create a Hadoop base node
* [Prepare Hadoop Nodes](./doc/md/Config-Hadoop-Nodes.md) : Create as many Hadoop nodes as required and set node specific values.
* [Install Ambari](./doc/md/Config-Ambari.md) : Install Ambari, which will install and manage the HDP cluster.
* [Configure Hadoop via Ambari](./doc/md/Config-Hadoop.md) : Set up Hadoop Services. 

### Kakfa
* [Configure Kafka](./doc/md/Config-Kafka.md) : Comments on Kafka configuration

## Zeppelin / R
* [Configure Baseline R Node](./doc/md/Config-BaseNode-Zeppelin.md) : Extend the Hadoop baseline to add support for R / Zeppelin
* [Configure Zeppelin](./doc/md/Config-Zeppelin.md) : Install Zeppelin via Ambari

## Elasticsearch
* [Create Baseline Elasticsearch Node](./doc/md/Config-BaseNode-ES.md) : Extend the baseline node to create an Elasticsearch base node.
* [Create Elasticsearch Cluster](./doc/md/Config-Elasticsearch.md) : Create Elasticsearch nodes and set node specific values. 

## Applications / Services
* [Create Containers for Applications](./doc/md/Config-BaseNode-App.md) : Create as many containers as needed for applications or other infrastructure services.

## Zabbix
* [Configure Zabbix Server](./doc/md/Config-Zappix.md) : Install and configure Zabbix for monitoring
* [Configure Zabbix Agent](./doc/md/Config-Zappix-Agent.md) : Install and configure Zabbix agent on nodes
