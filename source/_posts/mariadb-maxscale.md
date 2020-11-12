---
title: MariaDB MaxScale
date: 2020-03-06 15:29:58
categories: [Database]
tags: [Replication, MySQL, MaxScale]
---
# Intro

This blog is supposed to be used when someone else intends to have a simple MaxScale 2.4.7++ setup
considering a replication cluster. By replication cluster I mean one master and X slaves; in the 
case of this blog, I'm using just one slave, but, you can add as many as you want. 

There only 2 machines,
* 192.168.1.105, worked as master.
* 192.168.1.108, worked as slave, MaxScale also running on it.

<!--more-->
# Installation

Check the [official website](https://mariadb.com/downloads/#mariadb_platform-mariadb_maxscale) to get the rpm package in according with your system version.

```
[root@AEMG-CS zac]# wget https://dlm.mariadb.com/975790/MaxScale/2.4.7/centos/6/x86_64/maxscale-2.4.7-1.centos.6.x86_64.rpm
--2020-03-16 07:48:44--  https://dlm.mariadb.com/975790/MaxScale/2.4.7/centos/6/x86_64/maxscale-2.4.7-1.centos.6.x86_64.rpm
Resolving dlm.mariadb.com... 104.20.159.76, 104.20.158.76, 2606:4700:10::6814:9f4c, ...

[root@AEMG-CS zac]# rpm -ivh maxscale-2.4.7-1.centos.6.x86_64.rpm 
warning: maxscale-2.4.7-1.centos.6.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID 28c12247: NOKEY
Preparing...                ########################################### [100%]
   1:maxscale               ########################################### [100%]

[root@AEMG-CS zac]# rpm -q maxscale
maxscale-2.4.7-1.x86_64
[root@AEMG-CS zac]# maxscale -V
MaxScale 2.4.7 - f576680ed9062222f23ec9e6a3f0b23174ed0535
```

# Create MySQL Users

```sql
CREATE USER 'msadmin'@'%' IDENTIFIED BY 'password';
GRANT SELECT ON mysql.user TO 'msadmin'@'%';
GRANT SELECT ON mysql.db TO 'msadmin'@'%';
GRANT SELECT ON mysql.tables_priv TO 'msadmin'@'%';
GRANT SHOW DATABASES ON *.* TO 'msadmin'@'%';
SHOW GRANTS FOR msadmin;


CREATE USER 'msuser'@'192.168.86.%' IDENTIFIED BY 'password';
# For Slave
GRANT SELECT ON moodle.* TO 'msuser'@'192.168.86.%';
# For Master
GRANT SELECT, ON moodle.* TO 'msuser'@'192.168.86.%';

SHOW GRANTS FOR 'msuser'@'192.168.86.%';

```

# Split Read and Write

## maxscale.cnf
Copy the configuration file to maxscale.cnf.
```
vim /etc/maxscale.cnf
mkdir -p /tmp/maxscale
chown -R maxscale. /tmp/maxscale
```

``` bash
[maxscale]
threads=auto

[Splitter-Service]
type=service
router=readwritesplit
servers=node01, node02
user=maxscale
password=maxscale

[Splitter-Listener]
type=listener
service=Splitter-Service
protocol=MariaDBClient
port=4006
address=192.168.1.108

[node01]
type=server
address=192.168.1.105
port=3306
protocol=MariaDBBackend

[node02]
type=server
address=192.168.1.108
port=3306
protocol=MariaDBBackend

[MyMonitor]
type=monitor
module=mariadbmon
servers=node01,node02
user=maxscale
password=maxscale

[CLI]
type=service
router=cli

[CLI-Listener]
type=listener
service=CLI
protocol=maxscaled
address=localhost
port=6603
```

```
[root@localhost binlog]# systemctl start maxscale.service 
[root@localhost binlog]# maxctrl list servers
┌────────┬───────────────┬──────┬─────────────┬─────────────────┬──────┐
│ Server │ Address       │ Port │ Connections │ State           │ GTID │
├────────┼───────────────┼──────┼─────────────┼─────────────────┼──────┤
│ node01 │ 192.168.1.105 │ 3306 │ 0           │ Master, Running │      │
├────────┼───────────────┼──────┼─────────────┼─────────────────┼──────┤
│ node02 │ 192.168.1.108 │ 3306 │ 0           │ Running         │      │
└────────┴───────────────┴──────┴─────────────┴─────────────────┴──────┘
```
## Configure Replication in Mysql

### In Master Run SQL
``` sql
show master status
```

File|Position|Binlog_Do_DB|Binlog_Ignore_DB|Executed_Gtid_Set
---|---|---|---|---
mysql-bin.000006 | 346682 | replication_test,moodle_exclude||



### In Slave Run SQL
```sql
reset slave;
CHANGE MASTER TO MASTER_HOST='192.168.1.108',MASTER_PORT=5006,MASTER_USER='maxscale',MASTER_PASSWORD='maxscalepass',MASTER_LOG_FILE='mysql-bin.000006',MASTER_LOG_POS=346682;
start slave;
```
and then check the servers in MaxScale.
```
[root@localhost binlog]# maxctrl list servers
┌────────┬───────────────┬──────┬─────────────┬─────────────────┬──────┐
│ Server │ Address       │ Port │ Connections │ State           │ GTID │
├────────┼───────────────┼──────┼─────────────┼─────────────────┼──────┤
│ node01 │ 192.168.1.105 │ 3306 │ 0           │ Master, Running │      │
├────────┼───────────────┼──────┼─────────────┼─────────────────┼──────┤
│ node02 │ 192.168.1.108 │ 3306 │ 0           │ Slave, Running  │      │
└────────┴───────────────┴──────┴─────────────┴─────────────────┴──────┘
```

## Log Output of MaxScale is Valuable 

```
[root@localhost etc]# tail -f /tmp/maxscale/maxscale.log 
2020-03-06 15:52:17   info   : [MariaDBAuth] Added user: maxscale@% db: 'mysql' global: no
2020-03-06 15:52:17   notice : Listening for connections at [192.168.1.108]:4006
2020-03-06 15:52:17   notice : Service 'Splitter-Service' started (1/2)
2020-03-06 15:52:17   notice : Listening for connections at [localhost]:6603
2020-03-06 15:52:17   notice : Service 'CLI' started (2/2)
2020-03-06 15:52:17   notice : Loaded server states from journal file: /var/lib/maxscale/MyMonitor/monitor.dat
2020-03-06 15:52:17   notice : [mariadbmon] Selecting new master server.
2020-03-06 15:52:17   warning: [mariadbmon] 'node02' is not a valid master candidate because it is in read_only mode.
2020-03-06 15:52:17   notice : [mariadbmon] Setting 'node01' as master.
2020-03-06 15:52:17   notice : Server changed state: node01[192.168.1.105:3306]: new_master. [Running] -> [Master, Running]

2020-03-06 15:53:13   info   : Accept authentication from 'admin', using password. Request: /v1/servers
2020-03-06 15:53:13   info   : Accept authentication from 'admin', using password. Request: /v1/monitors/MyMonitor
2020-03-06 15:55:13   notice : Server changed state: node02[192.168.1.108:3306]: new_slave. [Running] -> [Slave, Running]
2020-03-06 15:56:01   info   : Accept authentication from 'admin', using password. Request: /v1/servers
2020-03-06 15:56:01   info   : Accept authentication from 'admin', using password. Request: /v1/monitors/MyMonitor
```

From the above log, we can see the slave server is read only, and it can only be slave.

## Login MaxScale from Mysql Command Line

``` sql
mysql -umaxscale -pmaxscalepass -h192.168.1.108 -P4006
```

**IMPORTANT NOTES**

Both master and slave should have this maxscale and identified by maxscalepass.

From the output log file, we can get the error if node2 doesn't have that account.
```
2020-03-06 16:11:00   info   : (1) [readwritesplit] Selected Slave: node02
2020-03-06 16:11:00   info   : (1) Started Splitter-Service client session [1] for 'zach' from 192.168.1.107
2020-03-06 16:11:00   info   : (1) Connected to 'node01' with thread id 15241
2020-03-06 16:11:00   info   : (1) Connected to 'node02' with thread id 455
2020-03-06 16:11:00   error  : (1) [mariadbbackend] Invalid authentication message from backend 'node02'. Error code: 1045, Msg : #28000Access denied for user 'zach'@'192.168.1.108' (using password: YES)
```

## Verify
``` sql
mysql> select @@hostname;
+-----------------------+
| @@hostname            |
+-----------------------+
| localhost.localdomain |
+-----------------------+
1 row in set (0.02 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.02 sec)

mysql> select @@hostname;
+-----------------+
| @@hostname      |
+-----------------+
| aemg-mel-server |
+-----------------+
1 row in set (0.01 sec)

mysql> rollback;
Query OK, 0 rows affected (0.01 sec)

mysql> select @@hostname;
+-----------------------+
| @@hostname            |
+-----------------------+
| localhost.localdomain |
+-----------------------+
1 row in set (0.01 sec)
```

```
2020-03-06 16:14:17   info   : (3) [MariaDBAuth] Client 'maxscale'@[192.168.1.107] is using an unsupported authenticator plugin 'caching_sha2_password'. Trying to switch to 'mysql_native_password'.
2020-03-06 16:14:17   info   : (3) [readwritesplit] Servers and router connection counts:
2020-03-06 16:14:17   info   : (3) [readwritesplit] current operations : 0 in 	[192.168.1.105]:3306 Master, Running
2020-03-06 16:14:17   info   : (3) [readwritesplit] current operations : 0 in 	[192.168.1.108]:3306 Slave, Running
2020-03-06 16:14:17   info   : (3) [readwritesplit] Connected to 'node01'
2020-03-06 16:14:17   info   : (3) [readwritesplit] Selected Master: node01
2020-03-06 16:14:17   info   : (3) [readwritesplit] Connected to 'node02'
2020-03-06 16:14:17   info   : (3) [readwritesplit] Selected Slave: node02
2020-03-06 16:14:17   info   : (3) Started Splitter-Service client session [3] for 'maxscale' from 192.168.1.107
2020-03-06 16:14:17   info   : (3) Connected to 'node01' with thread id 15297
2020-03-06 16:14:17   info   : (3) Connected to 'node02' with thread id 467
2020-03-06 16:14:17   info   : (3) > Autocommit: [enabled], trx is [not open], cmd: (0x03) COM_QUERY, plen: 37, type: QUERY_TYPE_READ|QUERY_TYPE_SYSVAR_READ, stmt: select @@version_comment limit 1 
2020-03-06 16:14:17   info   : (3) [readwritesplit] Route query to slave: node02 	[192.168.1.108]:3306 <
2020-03-06 16:14:17   info   : (3) [readwritesplit] Reply complete, last reply from node02

2020-03-06 16:23:08   info   : (3) > Autocommit: [enabled], trx is [not open], cmd: (0x03) COM_QUERY, plen: 23, type: QUERY_TYPE_READ|QUERY_TYPE_SYSVAR_READ, stmt: select @@host_name 
2020-03-06 16:23:08   info   : (3) [readwritesplit] Route query to slave: node02 	[192.168.1.108]:3306 <
2020-03-06 16:23:08   info   : (3) [readwritesplit] Pinging node01, idle for 530 seconds
2020-03-06 16:23:08   info   : (3) [readwritesplit] Reply complete, last reply from node02
2020-03-06 16:23:08   info   : (3) [mariadbbackend] Response to COM_CHANGE_USER is OK, writing stored query
2020-03-06 16:23:12   info   : (3) > Autocommit: [enabled], trx is [not open], cmd: (0x03) COM_QUERY, plen: 22, type: QUERY_TYPE_READ|QUERY_TYPE_SYSVAR_READ, stmt: select @@hostname 
2020-03-06 16:23:12   info   : (3) [readwritesplit] Route query to slave: node02 	[192.168.1.108]:3306 <
2020-03-06 16:23:12   info   : (3) [readwritesplit] Reply complete, last reply from node02

2020-03-06 16:23:51   info   : (3) > Autocommit: [enabled], trx is [open], cmd: (0x03) COM_QUERY, plen: 22, type: QUERY_TYPE_BEGIN_TRX, stmt: start transaction 
2020-03-06 16:23:51   info   : (3) [readwritesplit] Route query to master: node01 	[192.168.1.105]:3306 <
2020-03-06 16:23:51   info   : (3) [readwritesplit] Reply complete, last reply from node01

2020-03-06 16:24:13   info   : (3) > Autocommit: [enabled], trx is [open], cmd: (0x03) COM_QUERY, plen: 23, type: QUERY_TYPE_READ|QUERY_TYPE_SYSVAR_READ, stmt: select @@host_name 
2020-03-06 16:24:13   info   : (3) [readwritesplit] Route query to master: node01 	[192.168.1.105]:3306 <
2020-03-06 16:24:13   info   : (3) [readwritesplit] Reply complete, last reply from node01
2020-03-06 16:24:27   info   : (3) > Autocommit: [enabled], trx is [open], cmd: (0x03) COM_QUERY, plen: 22, type: QUERY_TYPE_READ|QUERY_TYPE_SYSVAR_READ, stmt: select @@hostname 
2020-03-06 16:24:27   info   : (3) [readwritesplit] Route query to master: node01 	[192.168.1.105]:3306 <
2020-03-06 16:24:27   info   : (3) [readwritesplit] Reply complete, last reply from node01

2020-03-06 16:25:06   info   : (3) > Autocommit: [enabled], trx is [open], cmd: (0x03) COM_QUERY, plen: 13, type: QUERY_TYPE_ROLLBACK, stmt: rollback 
2020-03-06 16:25:06   info   : (3) [readwritesplit] Route query to master: node01 	[192.168.1.105]:3306 <
2020-03-06 16:25:06   info   : (3) [readwritesplit] Reply complete, last reply from node01

2020-03-06 16:25:19   info   : (3) > Autocommit: [enabled], trx is [not open], cmd: (0x03) COM_QUERY, plen: 22, type: QUERY_TYPE_READ|QUERY_TYPE_SYSVAR_READ, stmt: select @@hostname 
2020-03-06 16:25:19   info   : (3) [readwritesplit] Route query to slave: node02 	[192.168.1.108]:3306 <
2020-03-06 16:25:19   info   : (3) [readwritesplit] Reply complete, last reply from node02
```

## Set Slave to READ-ONLY

To play save, avoid someone use command line accidentally update data in slave server, we'd better set slave to READ-ONLY.

# MaxScale Works As Binlog Server
``` bash
mkdir -p /data/binlog 
chown -R maxscale. /data/binlog
```

``` bash
[maxscale]
threads=4
log_info=1
logdir=/tmp/maxscale/

[Replication]
type=service
router=binlogrouter
user=maxscale
password=maxscalepass
#server_id=1251
#mariadb10-compatibility=1
#binlogdir=/var/lib/maxscale/
router_options=server_id=1251,heartbeat=30,binlogdir=/data/binlog,transaction_safety=1,mariadb10-compatibility=0,send_slave_heartbeat=1

[Replication-Listener]
type=listener
service=Replication
protocol=MariaDBClient
port=5006
address=192.168.1.108

[Splitter-Service]
type=service
router=readwritesplit
servers=node01, node02
user=maxscale
password=maxscale

[Splitter-Listener]
type=listener
service=Splitter-Service
protocol=MariaDBClient
port=4006
address=192.168.1.108

[node01]
type=server
address=192.168.1.105
port=3306
protocol=MariaDBBackend

[node02]
type=server
address=192.168.1.108
port=3306
protocol=MariaDBBackend

[MyMonitor]
type=monitor
module=mariadbmon
servers=node01,node02
user=maxscale
password=maxscale

[CLI]
type=service
router=cli

[CLI-Listener]
type=listener
service=CLI
protocol=maxscaled
address=localhost
port=6603
```

# Set Maintenance

Before you do anything to the slave node, you should put the slave into maintenance mode. This allows for planned, temporary removal of a database from the cluster within the need to change the MariaDB MaxScale configuration.
```
maxctrl set server server4 maintenance
# or
MaxScale> set server server4 maintenance
```
This will cause MariaDB MaxScale to stop routing any new requests to the server, however if there are currently requests executing on the server these will not be interrupted.

To bring the server back into service use the "clear server" command to clear the maintenance mode bit for that server.

```
maxctrl clear server server4 maintenance
# or
MaxScale> clear server server4 maintenance
```

# Starting the MaxScale Instance

## CentOS 6

<center>

Operation | Command
------- | -------
Start | service maxscale start
Stop | service maxscale stop
Status | service maxscale status
Restart | service maxscale restart
Enable startup | chkconfig --add maxscale
Disable startup | chkconfig --del maxscale

</center>
