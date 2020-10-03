---
title: XtraBackup
date: 2020-03-13 16:30:36
categories: 
tags: [MySQL, Replication]
---

This article is about the procedure to hot backup and configure a replication of MySQL sever.
The configuration servers are IP with 200 as master and 64 as slave;

# Preparation: 

It's a good practice to check your work station first before you do anything, you have to know as more as possible including the hardware and system limitation, so that you can get a rough idea when encountered with a problem.

``` sh
sudo su
cat /etc/centos-release
lscpu
ifconfig
nmcli device status #list network card
TheMaster$ ping 192.168.86.64
cat /etc/resolv.conf #DNS
cat /etc/sysconfig/network #Gateway
sestatus
firewall-cmd --list-all
sudo setenforce 0
ulimit -a # Check system limitation of open files number
ulimit -n 2000000 # Probably need to set open files limit
```
<!--more-->
# Master

As we're working on Database, disk space could be a much needed resource for some circumstances. Select the biggest disk space mount point.
```
df -h
mkdir -p /aemg/zac_db_repl_dir
```

Check if you got a available account.
```
mysql -uroot -p #Check you mysql account
> show grants;
> select @@datadir;
```

If it is a new MySQL sever, we should set up the database with the default password, by doing so, issue below commands to change root password.
```
grep "temporary password" /var/log/mysqld.log
SET PASSWORD = PASSWORD('mynewpass')
```
## Master MySQL Configure File

```
vim /etc/my.cnf
[mysqld]
general_log = 'OFF'
general_log_file=/usr/log/general.log
#bind-address = 0.0.0.0
server-id=200
log_bin=mysql-bin
log_error=mysql-bin.err
#binlog_do_db=moodle
```
```
nohup xtrabackup --defaults-file=/etc/my.cnf --backup --user=root --password= --target-dir=/aemg/zac_db_repl_dir &
xtrabackup --defaults-file=/etc/my.cnf --user=root --password=  --prepare --target-dir=/aemg/zac_db_repl_dir 
```
you can also use `compress` option to get a faster progress.
```
xtrabackup --backup --compress --compress-threads=4 --target-dir=/data/compressed/
xtrabackup --decompress --target-dir=/data/compressed/
xtrabackup --prepare --target-dir=/data/compressed/
````


# Slave

Select the biggest disk space mount point.

```
[root@CC64 lib]# df -h
devtmpfs                          24G     0   24G    0% /dev
tmpfs                             24G     0   24G    0% /dev/shm
tmpfs                             24G   25M   24G    1% /run
tmpfs                             24G     0   24G    0% /sys/fs/cgroup
/dev/mapper/centos-root           50G  3.9G   47G    8% /
/dev/mapper/centos-home          1.1T   40M  1.1T    1% /home
/dev/sda1                       1014M  174M  840M   18% /boot
192.168.86.199:/aemg/moodledata  6.2T  1.6T  4.3T   28% /aemg/moodledata
tmpfs                            4.7G     0  4.7G    0% /run/user/0
```

```
mkdir -p /home/mysqldatadir

ln -s /home/mysqldatadir /var/lib/mysql

TheMaster$ rsync -avpP -e ssh /aemg/zac_db_repl_dir 192.168.86.64:/home/zac

TheSlave$ mv /home/mysqldatadir/* /home/mysqldatadir_bak

TheSlave$ xtrabackup --move-back --target-dir=/home/zac/zac_db_repl_dir
```

If you don’t want to use any of the above options, you can additionally use rsync or cp to restore the files.
The datadir must be empty before restoring the backup. Also it’s important to note that MySQL server needs to be shut down.

```
TheSlave$ chown -R mysql:mysql /home/mysqldatadir

TheSlave$ systemctl restart mysqld

TheMaster|mysql> CREATE USER 'repl'@'%' IDENTIFIED BY 'password';

TheMaster|mysql> GRANT REPLICATION SLAVE ON *.*  TO 'repl'@'%' IDENTIFIED BY 'password';
```

login in master to test

```
TheSlave$ cat /var/lib/mysql/xtrabackup_binlog_info

TheSlave|mysql> CHANGE MASTER TO
                MASTER_HOST='$masterip',
                MASTER_USER='repl',
                MASTER_PASSWORD='$slavepass',
                MASTER_LOG_FILE='TheMaster-bin.000001',
                MASTER_LOG_POS=481;
TheSlave|mysql> START SLAVE;

TheSlave|mysql> SHOW SLAVE STATUS \G
```

# Problems

## File Descriptor Limitation

```
InnoDB: Operating system error number 24 in a file operation.
InnoDB: Error number 24 means 'Too many open files'
InnoDB: Some operating system error numbers are described at http://dev.mysql.com/doc/refman/5.7/en/operating-system-error-codes.html
InnoDB: File ./zf2tutorial/aemg_table_bangong_jiangli.ibd: 'open' returned OS error 124. Cannot continue operation
InnoDB: Cannot continue operation.
```

```
cat /proc/sys/fs/nr_open

find /aemg/data/ -name "*.ibd" | wc -l
1508
need to add another 1000

cat /proc/sys/fs/file-max

sysctl -w fs.file-max=21339338 

echo "fs.file-max=21339338" >> /etc/sysctl.conf

sysctl -w fs.nr_open=2000000

echo "fs.nr_open=2000000" >> /etc/sysctl.conf

ulimit -n 2000000 

ulimit -a
```

https://www.percona.com/blog/2016/12/28/using-percona-xtrabackup-mysql-instance-large-number-tables/


## File Exists

xtrabackup: Can't create/write to file 'trabackup_logfile' (Errcode: 17 - File exists)

Just delete it.

## [Warning] InnoDB: Table mysql/innodb_index_stats has length mismatch in the column name table_name.  Please run mysql_upgrade

```
mysqlcheck -uroot  -p --all-databases --check-upgrade --auto-repair
mysql_upgrade -uroot -p 
systemctl restart mysqld
```