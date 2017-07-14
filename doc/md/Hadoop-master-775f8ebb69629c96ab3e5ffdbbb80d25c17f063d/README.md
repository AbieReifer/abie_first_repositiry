# Hadoop Data Science Cluster #

This project documents the deployment and configuration of a Westat data science Hadoop cluster, based on the Hortonworks Data Platform (HDP) distribution of Hadoop and related technologies, using Ambari as the management agent.

Initially the data science cluster will only be used in production for the NHCS project. However, the test and staging environments may be used for any project considering a Hadoop cluster.

## [Deployment](./doc/md/Deployment.md) 
Cluster deployment, hosts and containers.  

## [Configuration](./doc/md/Configuration.md)  
Configuration of physical hosts and containers.

## Technologies ##

Initially the cluster will offer the following technologies, all of which are used in the NHCS project.

### Hadoop Ecosystem ###
* Ambari, the management system
* Zookeeper, the, well the zoo keeper, keeps track of everything and where everything is
* Hadoop Distributed File System (HDFS), the core data storage technology
* HBase, a distributed key-value pair data store
* Phoenix, a SQL interface to HBase
* Kafka, a message broker

The production cluster will also include:
* Ranger, access control across the cluster

Other Hadoop ecosystem technologies managed by Ambari can be added to the cluster on demand, such as MapReduce, Spark, Storm, etc.
The Test cluster now also has:
* Spark
* Zeppelin
* YARN / MapReduce
* Hive

### Elasticsearch ###
The cluster includes an Elasticsearch cluster, running entirely independently of the Hadoop cluster.  
The Elasticsearch cluster is initially running on 3 nodes.

## Data Integrity ##

### Data Redundancy ###
The cluster technologies (Hadoop and Elasticsearch) have been specifically designed for data redundancy.  

Both technologies (Hadoop and Elasticsearch) run on at least 3 nodes, with 2 replicas of all data. The 3 nodes of each cluster are
deployed across 3 different physical hosts. However, currently, all the hosts are in the same data center are subject to 
incidents which affect the entire data center.

### ZFS Bit Level Integrity ###
The HDFS data nodes, and the Elasticsearch indexes, are stored in ZFS Datasets (file systems). ZFS will provide bit level data integrity 
at the file system level. Each physical host has its own mirrored ZFS pool where these ZFS file systems are stored. 
Each container is also stored in its own ZFS file system, these latter file systems are created by LXD when the container is created.

### Data Recovery and Fallback ###
ZFS snapshots are taken of the ZFS file systems on a scheduled basis, default is 24 hours but these can be more frequent.
Off-site backups of the ZFS filesystems are planned.

## Cluster Recovery ##
There is no 'warm' cluster available in the event of a total cluster failure.

Total cluster 'fail-over' or recovery, is an outstanding issue.

Re-creating the current cluster; install of host OS and configuration and installation of the current cluster environment would take one day.




