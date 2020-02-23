---
title: New Load Balance Node Configuration
date: 2020-02-11 16:07:01
categories: [Linux]
tags:
---
Working on 204, inet is 26

# Prerequisite
```
sudo su
cat /etc/centos-release
lscpu
ifconfig
sestatus
sudo setenforce 0
```
<!--more-->

## User Configuration
You may need to add user to sudo group
```
adduser zac
passwd zac
usermod -aG wheel zac
chown zac:zac /aemg/moodle
```
# NFS
## NFS Server
```
vim /etc/exports
add
/aemg/moodledata        192.168.86.26(rw,async)
exportfs -ra
```
## NFS Client
```
df -a
yum list installed nfs-utils
yum install nfs-utils
mkdir -p /aemg/moodledata
chmod 777 /aemg/moodledata/
mount -t nfs 192.168.86.199:/aemg/moodledata /aemg/moodledata
vim /etc/fstab
append
192.168.86.199:/aemg/moodledata /aemg/moodledata        nfs rw
```

# Rsync

``` sh
yum list installed rsync
yum install rsync

# Check the firewall to see if it is listed(IMPORTANT)
firewall-cmd --list-services
firewall-cmd --permanent --zone=public --add-service=rsyncd
firewall-cmd --reload
```
Configuration of /etc/rsyncd.conf looks like below

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
```
# Create Secrets File
touch rsyncd.secrets
chmod 600 rsyncd.secrets

service xinetd start
```

## Sync code
```
rsync -azp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" /aemg/cloudclassroom/moodle zac@192.168.86.26::moodle --delete
```

## In server end add this to iwait.sh:
rsync -azvp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" --exclude "/aemg/cloudclassroom/moodle/config.php" /aemg/cloudclassroom/moodle zac@192.168.86.26::moodle --delete

nohup /bin/bash /home/zac/iwait.sh &


## xinetd
```
yum list installed xinetd
yum install xinetd
vim /etc/xinetd.d/rsync
```
add below code to `/etc/xinetd.d/rsync`
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

# Httpd & PHP

## Httpd
``` sh
yum --enablerepo=epel,remi install httpd
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```
## VirtualHost
```
ln -s /aemg/moodle /var/www/moodle
vim httpd/conf.d/cloudcampus.conf
```
``` yml
<VirtualHost *:80>
    DocumentRoot /var/www/moodle
    ServerName www4.cloudcampus.com.au
    ErrorLog logs/cloudcampus.com.au-error_log
    CustomLog logs/cloudcampus.com.au-access_log common
    #    Alias /moodle/ "/var/www/moodle/"
        <Directory "/var/www/moodle">
                AllowOverride All
        </Directory>
</VirtualHost>
```

## PHP
``` sh
yum install -y epel-release
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum install yum-utils
yum-config-manager --enable remi-php72
yum update
php -v
yum search php72 | more
yum install php php-fpm php-gd php-json php-mbstring php-mysqlnd php-xml php-xmlrpc php-opcache php-memcached php-zip php-curl php-mcrypt php-soap
# Or
yum -y --enablerepo=remi-php72 install php php-fpm php-gd php-json php-mbstring php-mysqlnd php-xml php-xmlrpc php-opcache php-memcached php-zip php-curl php-mcrypt php-soap
php --modules
php72 --modules
service httpd restart
service httpd reload

scp zac@192.168.86.65:/etc/php.ini /etc/php.ini.new
mv /etc/php.ini /etc/php.ini.bak
mv /etc/php.ini.new /etc/php.ini
systemctl restart httpd

vim /var/www/html/index.php
# Write and Test
<?php phpinfo(); ?>
```

# Add Node to Load Balancer

Configuration of Nginx

1. add node to upstream chain
2. add 192.168.86.26 www4.cloudcampus.com.au to `/etc/hosts`