---
title: MySQL Replication
date: 2020-02-26 12:52:31
categories: [Database]
tags: [MySQL, Replication]
---
# MySQL Replication Configuration
```
GRANT REPLICATION SLAVE ON *.* to 'rep1'@'192.168.8.11' identified by 'test123456';
FLUSH PRIVILEGES;
SHOW MASTER STATUS;
RESET MASTER;
FLUSH TABLES WITH READ LOCK;
UNLOCK TABLES;
```
<!--more-->

my.conf
```
[mysqld]
general_log = 'OFF'
general_log_file=/usr/log/general.log
#bind-address = 0.0.0.0
server-id=9579751
log_bin=mysql-bin
log_error=mysql-bin.err
binlog_do_db=replication_test
```
service mysqld restart

my.conf
```
server-id=1582587781
relay-log=/var/lib/mysql/mysql-relay-bin
relay-log-index=/var/lib/mysql/mysql-relay-bin.index
read-only=1
```

```
set global read_only=1;
show slave status\G;
change master to master_host='192.168.1.105',master_port=3306,master_user='slave108',master_password='slave108',master_log_file='mysql-bin.000001',master_log_pos=154;
start slave;
reset slave;     # 清除binlog 文件名及位置
reset slave all; # 清除slave 的连接配置信息
```
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```
```
stop slave; SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1; start slave;

mysqldump -uzach -p replication_test > /home/zac/mysql/replication_test.sql
scp /home/zac/mysql/replication_test.sql root@192.168.1.108:/home/

stat /home/replication_test.sql

create database cmdb default charset utf8;
mysql -uroot -p replication_test</home/replication_test.sql
```