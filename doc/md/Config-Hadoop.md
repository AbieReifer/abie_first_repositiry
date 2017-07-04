# Configure Hadoop via Ambari

Ambari should now be available on port 8080. 
The initial user name / password is  admin / admin

For example:  http://hdp-c20.westat.com:8080

## Create Cluster ##

Proceed to create the cluster.

When prompted, the base URLs for local repository are:  

HDP  http://hdp0.westat.com/hdp/HDP/ubuntu14/  
HDP-UTILS  http://hdp0.westat.com/hdp/HDP-UTILS-1.1.0.21/repos/ubuntu14/  

When prompted, the nodes to include in the cluster are (example is Test)
```shell
hdp-c21.westat.com
hdp-c22.westat.com
hdp-c23.westat.com
```

**Note on SSH key**
The following workaround for the key has worked.  
The key used by Ambari was generated in the Ambari container.
Display the key in the terminal, copy it into a text editor, such as Notepad++, then
remove all the line feeds (remove \n) and copy the remaining text into the
Ambari web interface where the key is requested.

```shell
# display the key
lxc exec ub1404-hdp0-tst -- /bin/bash
cat  ~/.ssh/id_rsa
```

### Trouble Shooting ###
If the nodes fail to register, back up to a previous step and make sure:
* containers have the correct hostname in /etc/hostname
* ownership of external storage location is 100000:100000

## Installing Services

After selecting the service(s) to install, use ```Customize Services``` to validate the
file system locations where services and data will be stored.

Keeping in mind the external storage locations mounted when preparing the Hadoop clusters 
make sure the external locations are chosen for data storage, this will generally be 
by preceding the proposed locations with /lxd/hdp; for example /lxd/hdp/hadoop/*

## Service Specific Configuration

### Kafka

*Maximum message size*  
Increase maximum message size to handle large payloads.
```
# fetch should be larger than message to avoid issues
message.max.bytes       = 10000000
replica.fetch.max.bytes = 10485760
```
*Number partitions*  
Set the number of partitions to something greater than the number of concurrent
consumers you expect to deploy. The number should be a multiple of the number of consumers.
```
num.partitions  = 10
```


