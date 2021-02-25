---
title: Deploy MySQL as RDS MySQL Slave
date: 2021-02-25 16:25:33
categories:
tags:
---
# Use Alibaba Cloud ECS to Build RDS MySQL Slave Database

Friendly reminder: The steps to install MySQL software will not be introduced in this article. After installing MySQL, you do not need to worry about initializing the database and starting the service. Just make sure that the installed MySQL version is not lower than the Alibaba Cloud RDS MySQL version, and create a new MySQL system user running the MySQL service in advance. For safety, part of the content has been mosaicked. The master-slave mode is determined by the version of RDS MySQL. This article mainly uses RDS MySQL version 5.6 as an example and uses the new GTID mode as the master and slave. The 5.5 version is more simple to configure the master and slave. The first 15 steps are the same. There are no 16 and 17 operations. The 18th step uses the traditional mode of binlog file and location as the master and slave. You can modify the corresponding SQL statement, so I won't add more explanation here.

Alibaba Cloud uses the open source Percona Xtrabackup tool to do full physical backup of RDS MySQL. Using Alibaba Cloud ECS self-built slave library still needs to use this tool to import full backup data. In order to solve the package dependency problem encountered during installation, it is recommended to install Percona Xtrabackup using yum, and it is recommended to install the latest version.

<!--more-->
## Download and Decompress the Entire MySQL Data Directory
``` bash
wget -c 'https://rdsbak-zb-v2.oss-cn-zhangjiakou.aliyuncs.com/custins501106879/data_20210225091140_qp.xb' -O data_0225_qp.xb
cat data_0225_qp.xb | xbstream -x -v -C /aemg/rds/mysqldata
innobackupex --decompress --remove-original /aemg/rds/mysqldata
innobackupex --defaults-file=/aemg/rds/mysqldata/backup-my.cnf --apply-log /aemg/rds/mysqldata
ln -s /aemg/rds/mysqldata /var/lib/mysql
chown -R mysql:mysql /aemg/rds/mysqldata
```

## Change the root Password
``` bash
mysqld_safe --skip-grant-tables &
mysql

select user from mysql.user
drop trigger sys.sys_config_insert_set_user;
drop trigger sys.sys_config_update_set_user;
find /var/lib/mysql/mysql *.trg
rm *.trg
UPDATE mysql.user SET authentication_string=PASSWORD('YOUR_PASSWORD') WHERE User='aliyun_root';

mysqladmin -u aliyun_root -p shutdown
service mysqld start
```

## Clear the Database

``` sql
mysql -ualiyun_root -p -h127.0.0.1

RESET MASTER;
RESET SLAVE;
TRUNCATE TABLE mysql.slave_relay_log_info;
TRUNCATE TABLE mysql.slave_master_info;
TRUNCATE TABLE mysql.slave_worker_info;
```

## Edit my.cnf
``` yml
master-info-repository=file
relay-log-info_repository=file
binlog-format=ROW
gtid-mode=on
enforce-gtid-consistency=true
```

## Setup Slave Configuration
``` sql
cat xtrabackup_slave_info
# Copy set global and run in MySQL cmd
STOP SLAVE;
SET GLOBAL gtid_purged='526087d7-6aac-11eb-9989-0c42a1f02216:1-776130, da03d60b-fbae-11ea-b946-0c42a141c25a:1-1945178';
CHANGE MASTER TO MASTER_HOST='rm.rds.aliyuncs.com', MASTER_PORT=3306, MASTER_USER='root', MASTER_PASSWORD='YOUR_PASSWORD', MASTER_AUTO_POSITION=1;
START SLAVE;
SHOW SLAVE STATUS \G
```

## Ref

[Ali Official](https://help.aliyun.com/knowledge_detail/41817.html?spm=a2c4g.11186623.6.765.7c0264177YqLoL)

[Developer Doc](https://yq.aliyun.com/articles/716101)


