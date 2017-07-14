# Install and Configure Zabbix

## Postgres Config

Switch to user postgres
```
sudo -u postgres
```

As user postgres connect to databases
```
psql
```
In psql terminal

Create user zabbix
Create database zabbix
Give user zabbix all privileges on database zabbix
```
CREATE USER zabbix PASSWORD zabbix;
CREATE DATABASE zabbix;
GRANT ALL ON DATABASE zabbix TO zabbix;
```
Exit psql
Exit user postgres

To connect as user zabbix, use IP address
```
zcat /usr/share/doc/zabbix-server-pgsql/create.sql.gz | psql -U zabbix -h 127.0.0.1
```
## Zabbix server config
```shell
nano /etc/zabbix/zabbix_server.conf

### Option: DBHost
#       Database host name.
#       If set to localhost, socket is used for MySQL.
#       If set to empty string, socket is used for PostgreSQL.
#
# Mandatory: no
# Default:
# DBHost=localhost
DBHost=127.0.0.1


### Option: DBName
#       Database name.
#       For SQLite3 path to database file must be provided. DBUser and DBPassword are ignored.
#
# Mandatory: yes
# Default:
# DBName=
DBName=zabbix

### Option: DBSchema
#       Schema name. Used for IBM DB2 and PostgreSQL.
#
# Mandatory: no
# Default:
# DBSchema=

### Option: DBUser
#       Database user. Ignored for SQLite.
#
# Mandatory: no
# Default:
# DBUser=
DBUser=zabbix

### Option: DBPassword
#       Database password. Ignored for SQLite.
#       Comment this line if no password is used.
#
# Mandatory: no
# Default:
# DBPassword=
DBPassword=zabbix

### Option: DBSocket
#       Path to MySQL socket.
#
# Mandatory: no
# Default:
# DBSocket=/tmp/mysql.sock

### Option: DBPort
#       Database port when not using local socket. Ignored for SQLite.
#
# Mandatory: no
# Range: 1024-65535
# Default (for MySQL):
DBPort=5432

```


## Front End
See https://www.zabbix.com/forum/showthread.php?t=45733

Using the repository install method, found issues installing Zabbix 3.2 on Ubuntu 16.04.

Using this guide:
https://www.zabbix.com/documentation..._zabbix_server

### PHP config

Zabbix installation guide is incomplete for Ubuntu and Postgres.

When setting the time zone, make sure you do it for both versions of PHP.

``` shell
nano /etc/apache2/conf-enabled/zabbix.conf

# php_value date.timezone Europe/Riga
php_value date.timezone America/New_York
```

You must also install the following additional packages:
```
sudo apt-get install php-bcmath
sudo apt-get install php-mbstring
sudo apt-get install php-xml
sudo apt-get install php5-pgsql
```
If not you will get errors when the front-end configuration verifies the installation.

After this is done, proceed restart Apache and point browser to UI interface.
Restart apache:
``` shell
sudo service apache2 restart
```

## UI Verification 
Log into the system.
http://10.1.245.49/zabbix

Admin/zabbix
