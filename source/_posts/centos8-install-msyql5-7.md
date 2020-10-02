---
title: Centos8 Install MySQL 5.7
date: 2020-06-03 09:47:53
categories: [Database, Linux]
tags: [MySQL]
---
# Normal Way

<!--more-->

```
sudo dnf remove @mysql
sudo dnf module reset mysql && sudo dnf module disable mysql
sudo vi /etc/yum.repos.d/mysql-community.repo

[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=0

[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/7/$basearch/
enabled=1
gpgcheck=0

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/7/$basearch/
enabled=1
gpgcheck=0

sudo dnf --enablerepo=mysql57-community install mysql-community-server
rpm -qi mysql-community-server
sudo systemctl enable --now mysqld.service
```

# Other image

``` bash
vi /etc/yum.repos.d/mysql-community.repo

# Copy and Paste below origins

[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-connectors-community-el7-$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-tools-community-el7-$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-5.6-community]
name=MySQL 5.6 Community Server
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-5.6-community-el7-$basearch/
enabled=0
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql

[mysql-5.7-community]
name=MySQL 5.7 Community Server
baseurl=https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-5.7-community-el7-$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.mysql.com/RPM-GPG-KEY-mysql
```

Use below command to install
``` bash
# mysql-5.7-community
dnf --enablerepo=mysql-5.7-community install mysql-community-server
```

[tsinghua image](https://mirror.tuna.tsinghua.edu.cn/help/mysql/)