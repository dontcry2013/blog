---
title: yum repo
date: 2020-03-12 16:40:09
categories: [Linux]
tags: [CentOS]
---

A problem happened when I try to install percona-xtrabackup-24, according to the official [document](https://www.percona.com/doc/percona-xtrabackup/2.4/installation/yum_repo.html), in order to sucessfully install Percona XtraBackup on CentOS prior to version 7, the libev package needs to be installed first. This libev package can be installed from the EPEL repositories.

<!--more-->

```
[root@AEMG-CS zac]# wget https://repo.percona.com/yum/percona-release-latest.noarch.rpm
--2020-03-12 10:33:50--  https://repo.percona.com/yum/percona-release-latest.noarch.rpm
Resolving repo.percona.com... 167.99.233.229, 167.71.118.3, 157.245.119.64
Connecting to repo.percona.com|167.99.233.229|:443... connected.
HTTP request sent, awaiting response... 

200 OK

[root@AEMG-CC zac]# ls
epel-release-6-8.noarch.rpm  inotify_md.sh  iwait.sh  nohup.out

[root@AEMG-CC zac]# yum install epel-release-6-8.noarch.rpm 
Loaded plugins: fastestmirror, security
Setting up Install Process
Examining epel-release-6-8.noarch.rpm: epel-release-6-8.noarch
epel-release-6-8.noarch.rpm: does not update installed package.
Error: Nothing to do

[root@AEMG-CC zac]# yum install libev
Loaded plugins: fastestmirror, security
Setting up Install Process
Repository base is listed more than once in the configuration
Repository updates is listed more than once in the configuration
Repository extras is listed more than once in the configuration
Repository centosplus is listed more than once in the configuration
Repository contrib is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * atomic: www6.atomicorp.com
 * remi-php55: mirrors.tuna.tsinghua.edu.cn
 * remi-php72: mirrors.tuna.tsinghua.edu.cn
 * remi-safe: mirrors.tuna.tsinghua.edu.cn
 * webtatic: uk.repo.webtatic.com
No package libev available.
Error: Nothing to do
```
From the above log, we can see it is no good to install libev.

Check with `yum repolist` respectively to see if there are differences in two servers, we get:

* The one we're currently working on:

![repolist][repolist]
  
* The one which previously setup: 

![repolist][cs-repolist]

After the comparison, I can find easily that epel repo is missing. Copy the repo base url from the working server which called CS, add it to the server CC. Everyhing works fine!

```
[root@AEMG-CS zac]# yum -v repolist

...

Repo-id      : epel
Repo-name    : Extra Packages for Enterprise Linux 6 - x86_64
Repo-revision: 1543548839
Repo-updated : Fri Nov 30 11:56:51 2018
Repo-pkgs    : 12,501
Repo-size    : 11 G
Repo-metalink: https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=x86_64
  Updated    : Fri Nov 30 11:56:51 2018
Repo-baseurl : http://mirror.horizon.vn/epel/6/x86_64/ (17 more)
Repo-expire  : 21,600 second(s) (last: Fri Nov 30 19:30:55 2018)

...

```

```
[root@AEMG-CC zac]# yum-config-manager --add-repo http://mirror.horizon.vn/epel/6/x86_64/
Loaded plugins: fastestmirror
Repository base is listed more than once in the configuration
Repository updates is listed more than once in the configuration
Repository extras is listed more than once in the configuration
Repository centosplus is listed more than once in the configuration
Repository contrib is listed more than once in the configuration
adding repo from: http://mirror.horizon.vn/epel/6/x86_64/

[mirror.horizon.vn_epel_6_x86_64_]
name=added from: http://mirror.horizon.vn/epel/6/x86_64/
baseurl=http://mirror.horizon.vn/epel/6/x86_64/
enabled=1y

[root@AEMG-CC zac]# yum install libev
Installed:
  libev.x86_64 0:4.03-3.el6  

[root@AEMG-CC zac]# yum install percona-xtrabackup-24
Installed:
  percona-xtrabackup-24.x86_64 0:2.4.18-1.el6

Complete!

[root@AEMG-CC zac]# which innobackupex
/usr/bin/innobackupex
```
# Check Version

## Check if Package Exists
```
[root@localhost ~]# yum search maxscale
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.as24220.net
 * epel: mirror.as24220.net
 * extras: mirror.as24220.net
 * remi-php72: remi.mirror.digitalpacific.com.au
 * remi-safe: remi.mirror.digitalpacific.com.au
 * updates: mirror.intergrid.com.au
============================================================================================ N/S matched: maxscale =============================================================================================
maxscale.x86_64 : MaxScale - An intelligent database proxy
```

## Check the Version Will be Installed
```
[root@localhost ~]# yum info maxscale
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.as24220.net
 * epel: mirror.as24220.net
 * extras: mirror.as24220.net
 * remi-php72: remi.mirror.digitalpacific.com.au
 * remi-safe: remi.mirror.digitalpacific.com.au
 * updates: mirror.intergrid.com.au
Installed Packages
Name        : maxscale
Arch        : x86_64
Version     : 2.4.7
Release     : 1
Size        : 119 M
Repo        : installed
Summary     : MaxScale - An intelligent database proxy
License     : MariaDB BSL 1.1
Description : 
            : The MariaDB Corporation MaxScale is an intelligent proxy that allows forwarding of
            : database statements to one or more database servers using complex rules,
            : a semantic understanding of the database statements and the roles of
            : the various servers within the backend cluster of databases.
            : 
            : MaxScale is designed to provide load balancing and high availability
            : functionality transparently to the applications. In addition it provides
            : a highly scalable and flexible architecture, with plugin components to
            : support different protocols and routing decisions.
```

## Check Local Installed Version

```
[root@localhost ~]# rpm -q maxscale
maxscale-2.4.7-1.x86_64
```


[repolist]: /blog/img/repolist.jpg "repolist"
[cs-repolist]: /blog/img/cs-repolist.jpg "cs-repolist"