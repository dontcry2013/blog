---
title: inotify
date: 2020-01-24 10:18:39
categories:
tags:
---
# 

<!--more-->


``` bash
rpm -qa | grep gcc
rpm -qa | grep make
# Or
yum list installed gcc 
yum list installed make
# Or
yum install gcc make

wget -c https://github.com/rvoicilas/inotify-tools/archive/3.20.1.tar.gz

tar xvzf 3.20.1.tar.gz 

./configure && make && make install
 
chmod 600 rsyncd.secrets

# 199
[root@AEMG-CC zac]# rsync -avzP --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" zac@192.168.86.65::test /home/zac/test  --delete

```