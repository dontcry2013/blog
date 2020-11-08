---
title: Docker Deploy MySQL
date: 2020-10-05 23:34:31
categories: [Virtualization]
tags: [Docker, MySQL]
---
# Installation

<!--more-->


``` bash
docker search mysql
docker pull mysql:latest
docker ps -a
docker run -itd --name mysql8 -v /Users/devmac/my.cnf:/etc/mysql/my.cnf -v /opt/mysqldir:/opt/mysqldir mysql:8.0.21
docker logs $(docker ps -a | grep mysql8 | cut -d' ' -f 1)
docker exec -it mysql8 bash

docker cp mysql8:/etc/my.cnf .
(correct the file)
docker cp my.cnf mysql8:/etc/my.cnf

docker run \
--detach \
--name=mysql8 \
--publish 3306:3306 \
--volume=/Users/devmac/my.cnf:/etc/mysql/my.cnf \
--volume=/opt/mysqldir:/opt/mysqldir mysql:8.0.21
```
For the previous example, I should put a port mapping in the commnad, which goes like this:
``` bash
docker run -itd --name mysql8 -v /Users/devmac/my.cnf:/etc/mysql/my.cnf -v /opt/mysqldir:/opt/mysqldir --publish 3306:3306 mysql:8.0.21
```
If you want to initialize the database with root password, go like this:
```
docker run -itd --name mysql8 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```
## my.cnf in /Users/devmac/my.cnf
```
[mysqld]
character-set-server = utf8mb4
disable_log_bin
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE
max_allowed_packet=256M
innodb_log_file_size=256M
#innodb_data_file_path=/var/lib/mysql/ibdata1:768M:autoextend
wait_timeout=28800
#lower_case_table_names=1
datadir=/opt/mysqldir
secure-file-priv=NULL                   
```
# Docker Change Port Mapping for an Existing Container
Most common answer’s a roundabout solution.
* You cannot do that! You’ll have to create a new container with proper port mapping
* Create an image of your existing container. Then use “docker run” with your desired port mapping for your “newly created” image.

> Step 1: Using “docker inspect” get details about current port mapping. This will be seen under “NetworkSettings”. And “PortBindings” under “HostConfig”.

> Step 2: Stop the container and docker engine before editing the below files. Go to `/var/lib/docker`

```
$ vi /var/lib/docker/containers/[hash_of_the_container]/config.v2.json
...
{
"Config": {
....
    "ExposedPorts": {
        "3306/tcp": {},
    },
....
},
"NetworkSettings": {
....
    "Ports": {
        "3306/tcp": [
            {
                "HostIp": "",
                "HostPort": "3306"
            }
        ],
    },
....
}
```
```
$ vi /var/lib/docker/containers/[hash_of_the_container]/hostconfig.json
{
....
 "PortBindings": {
    "3306/tcp": [
        {
            "HostIp": "",
            "HostPort": "3306"
        }
    ],
 },
.....
}
```
>Step 3: Re-start your docker engine (docker service via systemctl). Verify docker engine has started successfully, without any errors. Start your container.

# Where is /var/lib/docker on Mac/OS X

```
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
ctrl + a + d
screen -ls
screen -X quit
```
I suggest to use `cat` to copy, edit in you VScode, and `vi` to delete original one, post the modify text. It's much convenient compared with just edit in `vi`.


# Connect to MySQL

open another shell to test the connection to MySQL.
```
mysql -uroot -p -h127.0.0.1

mysql> SHOW FULL TABLES IN ss WHERE TABLE_TYPE LIKE 'VIEW';
mysql> SHOW CREATE VIEW v_teaching_signations\G

mysql> mysql -uUSER -p -e 'SHOW VARIABLES WHERE Variable_Name LIKE "%dir"'

mysql> use mysql;
mysql> SELECT * FROM user\G
mysql> SELECT user();
mysql> SELECT user, host, db, command FROM information_schema.processlist;
```

## Check Which MySQL Configuration File is Used
```
$ /usr/sbin/mysqld --verbose --help | grep -A 1 "Default options"
Default options are read from the following files in the given order:
/etc/mysql/my.cnf ~/.my.cnf /usr/etc/my.cnf
```