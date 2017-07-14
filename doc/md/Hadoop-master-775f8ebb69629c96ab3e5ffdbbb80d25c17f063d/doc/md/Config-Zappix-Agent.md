# Configure Zappix Agent

## Install Agent

```shell
# on the package cache machine, as of writing this was on host hdp1
sudo wget http://repo.zabbix.com/zabbix/3.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.2-1+xenial_all.deb
sudo dpkg -i zabbix-release_3.2-1+xenial_all.deb
sudo apt-get update

# on containers
sudo apt-get install zabbix-agent
```
## Configure
On each container, delete existing file contents and replace with following.
This should be part of the base container.

```shell
sudo rm /etc/zabbix/zabbix_agentd.conf
sudo nano /etc/zabbix/zabbix_agentd.conf


# Zabbix Agent Config file

PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix-agent/zabbix_agentd.log
LogFileSize=1
EnableRemoteCommands=1
Server=10.1.245.49
ServerActive=10.1.245.49
## Set this to the host name of the machine running the agent
Hostname=hdp-cXX.westat.com
```

## Start Agent

```shell
 sudo service zabbix-agent stop
 sudo service zabbix-agent start
```


For reference this is the original file with comments describing parameters.

```shell

############ GENERAL PARAMETERS #################

### Option: PidFile
# Default:
# PidFile=/tmp/zabbix_agentd.pid

PidFile=/var/run/zabbix/zabbix_agentd.pid

### Option: LogFile
#       Name of log file.
#       If not set, syslog is used.
LogFile=/var/log/zabbix-agent/zabbix_agentd.log

### Option: LogFileSize
# Default:
# LogFileSize=1

### Option: DebugLevel
# DebugLevel=3

### Option: SourceIP
#       Source IP address for outgoing connections.
# Default:
# SourceIP=

### Option: EnableRemoteCommands
#       Whether remote commands from Zabbix server are allowed.
#       0 - not allowed
#       1 - allowed
EnableRemoteCommands=1

### Option: LogRemoteCommands
#       Enable logging of executed shell commands as warnings.
#       0 - disabled
# LogRemoteCommands=0

##### Passive checks related

### Option: Server
#       List of comma delimited IP addresses (or hostnames) of Zabbix servers.
Server=10.1.245.49

### Option: ListenPort
# Default:
# ListenPort=10050

### Option: ListenIP
#       List of comma delimited IP addresses that the agent should listen on.
#       First IP address is sent to Zabbix server if connecting to it to retrieve list of active checks.
# Default:
# ListenIP=0.0.0.0

### Option: StartAgents
#       Number of pre-forked instances of zabbix_agentd that process passive checks.
# Default:
# StartAgents=3

##### Active checks related

### Option: ServerActive
#       List of comma delimited IP:port (or hostname:port) pairs of Zabbix servers for active checks.
# Default:
ServerActive=10.1.245.49

### Option: Hostname
#       Unique, case sensitive hostname.
#       Required for active checks and must match hostname as configured on the server.
# This is the hostname of the machine on which the agent is running
Hostname=hdp0.westat.com

### Option: HostnameItem
#       Item used for generating Hostname if it is undefined. Ignored if Hostname is defined.
# Default:
# HostnameItem=system.hostname

## ALL OTHER PARAMETERS CAN BE LEFT AS IS IN DEFAULT CONFIG
```



