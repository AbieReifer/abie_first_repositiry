## Deployment ##

The cluster is deployed on 3 physical hosts. Each host has 2 CPUs * 6 cores, for a total of 36 cores across the cluster.
Each host has 128GB of RAM for a total of 384GB across the cluster.

The cluster offers 3 environments, the URL is the Ambari console.
* Dev/Test   http://hdp-c20.westat.com:8080
* Staging    http://hdp-c30.westat.com:8080
* Production http://hdp-c40.westat.com:8080
* Dev/Test2   http://hdp-c50.westat.com:8080

All nodes in all clusters are created in separate Linux containers, managed by LXD.  Initially 30 containers have been deployed, 10 per environment.  
Additional containers can be created on demand - the only constraint is the availability of IP addresses (and overall cluster RAM).

Currently the 10 IP addresses assigned to each environment are allocated as follows:
+ 4 for Hadoop: 1 for Ambari (management) and 3 data nodes
+ 3 for Elasticsearch
+ 2 as application containers
+ 1 spare.  

![Test_Cluster](/doc/img/DataScienceCluster.png?raw=true "Test Cluster Overview")

The NHCS project will be running 2 instances of a number of its micro-services, one on each of the 2 application containers.

Containers can be moved from host to host, however the initial deployment is as follows.

### Hosts and Containers ###
Naming conventions;
Container DNS names are tied to their IP addresses (at least initially).
Container LXD names reflect their base OS image and their function. LXD names are unique across all hosts, as the containers may be moved between hosts.
* Hosts: ```hdp<integer>```
* Container DNS: ```hdp-c<last stanza of IP>```
* Container Name (LXD name): ```<OS>-<cluster><integer>-<environment>```

#### Initial allocation of containers across hosts ####

|**Env**	|**Allocation**	|**HostIP**	|**HostDNS**	|**C-IP**	|**C-DNS**	|**C-LXD-Name**
|---	|---	|---	|---	|---	|---	|---
|Test	|HDP-Ambari	|10.1.245.10	|hdp0	|10.1.245.20	|hdp-c20.westat.com	|ub1404-hdp0-tst
|Test	|HDP-Node1	|10.1.245.11	|hdp1	|10.1.245.21	|hdp-c21.westat.com	|ub1404-hdp1-tst
|Test	|HDP-Node2	|10.1.245.12	|hdp2	|10.1.245.22	|hdp-c22.westat.com	|ub1404-hdp2-tst
|Test	|HDP-Node3	|10.1.245.10	|hdp0	|10.1.245.23	|hdp-c23.westat.com	|ub1404-hdp3-tst
|Test	|ES-Node0	|10.1.245.10	|hdp0	|10.1.245.24	|hdp-c24.westat.com	|ub1604-es0-tst
|Test	|ES-Node1	|10.1.245.11	|hdp1	|10.1.245.25	|hdp-c25.westat.com	|ub1604-es1-tst
|Test	|ES-Node2	|10.1.245.12	|hdp2	|10.1.245.26	|hdp-c26.westat.com	|ub1604-es2-tst
|Test	|Services	|10.1.245.11	|hdp1	|10.1.245.27	|hdp-c27.westat.com	|ub1604-app01-tst
|Test	|Redis / Services	|10.1.245.12	|hdp2	|10.1.245.28	|hdp-c28.westat.com	|ub1604-app02-tst
|Test	|Services	|10.1.245.11	|hdp1	|10.1.245.29	|hdp-c29.westat.com	|ub1604-app03-tst
|Staging	|HDP-Ambari	|10.1.245.10	|hdp0	|10.1.245.30	|hdp-c30.westat.com	|ub1404-hdp0-stg
|Staging	|HDP-Node1	|10.1.245.11	|hdp1	|10.1.245.31	|hdp-c31.westat.com	|ub1404-hdp1-stg
|Staging	|HDP-Node2	|10.1.245.12	|hdp2	|10.1.245.32	|hdp-c32.westat.com	|ub1404-hdp2-stg
|Staging	|HDP-Node3	|10.1.245.10	|hdp0	|10.1.245.33	|hdp-c33.westat.com	|ub1404-hdp3-stg
|Staging	|ES-Node0	|10.1.245.10	|hdp0	|10.1.245.34	|hdp-c34.westat.com	|ub1604-es0-stg
|Staging	|ES-Node1	|10.1.245.11	|hdp1	|10.1.245.35	|hdp-c35.westat.com	|ub1604-es1-stg
|Staging	|ES-Node2	|10.1.245.12	|hdp2	|10.1.245.36	|hdp-c36.westat.com	|ub1604-es2-stg
|Staging	|Services	|10.1.245.11	|hdp1	|10.1.245.37	|hdp-c37.westat.com	|ub1604-app01-stg
|Staging	|Redis / Services	|10.1.245.12	|hdp2	|10.1.245.38	|hdp-c38.westat.com	|ub1604-app02-stg
|Staging	|Services	|10.1.245.11	|hdp1	|10.1.245.39	|hdp-c39.westat.com	|ub1604-app03-stg
|Production	|HDP-Ambari	|10.1.245.10	|hdp0	|10.1.245.40	|hdp-c40.westat.com	|ub1404-hdp0-prd
|Production	|HDP-Node1	|10.1.245.11	|hdp1	|10.1.245.41	|hdp-c41.westat.com	|ub1404-hdp1-prd
|Production	|HDP-Node2	|10.1.245.12	|hdp2	|10.1.245.42	|hdp-c42.westat.com	|ub1404-hdp2-prd
|Production	|HDP-Node3	|10.1.245.10	|hdp0	|10.1.245.43	|hdp-c43.westat.com	|ub1404-hdp3-prd
|Production	|ES-Node0	|10.1.245.10	|hdp0	|10.1.245.44	|hdp-c44.westat.com	|ub1604-es0-prd
|Production	|ES-Node1	|10.1.245.11	|hdp1	|10.1.245.45	|hdp-c45.westat.com	|ub1604-es1-prd
|Production	|ES-Node2	|10.1.245.12	|hdp2	|10.1.245.46	|hdp-c46.westat.com	|ub1604-es2-prd
|Production	|Services	|10.1.245.11	|hdp1	|10.1.245.47	|hdp-c47.westat.com	|ub1604-app01-prd
|Production	|Redis / Services	|10.1.245.12	|hdp2	|10.1.245.48	|hdp-c48.westat.com	|ub1604-app02-prd
|Production	|Zabbix	|10.1.245.11	|hdp1	|10.1.245.49	|hdp-c49.westat.com	|ub1604-zabbix
|	|	|	|	|	|	|
|Test2	|HDP-Ambari	|10.1.245.10	|hdp0	|10.1.245.50	|hdp-c50.westat.com	|ub1604-hdp0-tst
|Test2	|HDP-Node1	|10.1.245.11	|hdp1	|10.1.245.51	|hdp-c51.westat.com	|ub1604-hdp1-tst
|Test2	|HDP-Node2	|10.1.245.12	|hdp2	|10.1.245.52	|hdp-c52.westat.com	|ub1604-hdp2-tst
|Test2	|HDP-Node3	|10.1.245.10	|hdp0	|10.1.245.53	|hdp-c53.westat.com	|ub1604-hdp3-tst

