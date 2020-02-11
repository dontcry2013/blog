---
title: new-node-nfs-rsync
date: 2020-02-11 16:07:01
categories:
tags:
---
Working on 204, inet is 26

# Prerequisite
sudo su
cat centos-release
lscpu
ifconfig
sestatus
sudo setenforce 0

<!--more-->

# NFS Server
vim /etc/exports
add
/aemg/moodledata        192.168.86.26(rw,async)
exportfs -ra

# NFS Client
df -a
yum list installed nfs-utils
yum install nfs-utils
mkdir -p /aemg/moodledata
chmod 777 /aemg/moodledata/
mount -t nfs 192.168.86.199:/aemg/moodledata /aemg/moodledata
vim /etc/fstab
append
192.168.86.199:/aemg/moodledata /aemg/moodledata        nfs rw

# xinetd
yum list installed xinetd
yum install xinetd
vim xinetd.d/rsync
```
service rsync
 {
     port            = 873
     disable         = no
     socket_type     = stream
     protocol        = tcp
     wait            = no
     user            = root
     group           = root
     groups          = yes
     server          = /usr/bin/rsync
     server_args = --daemon --config /etc/rsyncd.conf
 }
```

# Rsync
yum list installed rsync
yum install rsync
```
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsync.log
address = 192.168.86.26
port = 873
secrets file=/etc/rsyncd.secrets
uid = 0
gid = 0
use chroot = yes
read only = no
write only = no
hosts allow = 192.168.86.0/24
hosts deny = *
max connections = 5
transfer logging = yes
log format = %t %a %m %f %b
syslog facility = local3
timeout = 300

[moodle]
path = /aemg
list=no
auth users = zac
exclude = moodle/config.php
```
touch rsyncd.secrets
chmod 600 rsyncd.secrets

service xinetd start

## Sync code
rsync -azp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" /aemg/cloudclassroom/moodle zac@192.168.86.26::moodle --delete

## Add User to Sudo group
adduser zac
passwd zac
usermod -aG wheel zac
chown zac:zac /aemg/moodle

## In server end add this to iwait.sh:
rsync -azvp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" --exclude "/aemg/cloudclassroom/moodle/config.php" /aemg/cloudclassroom/moodle zac@192.168.86.26::moodle --delete

nohup /bin/bash /home/zac/iwait.sh &
