---
title: New Load Balance Node Configuration
date: 2020-02-11 16:07:01
categories: [Linux]
tags:
---
Working on 204, inet is 26

# Prerequisite
``` sh
sudo su
cat /etc/centos-release
uname -a # Linux Core Version
ulimit -a
lscpu
lsof -i # To display active TCP and UDP endpoints
ifconfig
nmcli device status #list network card
ls /etc/sysconfig/network-scripts/ #network card scripts
cat /etc/resolv.conf #DNS
cat /etc/sysconfig/network #Gateway
sestatus
sudo setenforce 0
which telnet
date
route #check route ip table
```
<!--more-->

>Note: Following configuration of firewall used many services, instead of assign port directly, using service is more straightforward, check the service-port map firstly.
```
vim /etc/services
```

>Note: Check the system clock would be very important, since web server relies on session time, a wrong time in server could result in login problems.
```
# Check timezone
timedatectl

# If wrong, set the timezone
timedatectl list-timezones |  grep  -E "Asia/S.*"
timedatectl set-timezone Asia/Shanghai

# Sync clock
firewall-cmd --permanent --zone=public --add-service=ntp
firewall-cmd --reload

systemctl start chronyd.service
systemctl enable chronyd.service
chronyc -a makestep
```
>Note: To disable selinux permanently, we should change the configuration file. Mysqld sometimes requires SELinux to be disabled.
``` bash
vim /etc/selinux/config
# Set SELINUX=disabled
sudo shutdown -r now
```

## User Configuration
You may need to add user to sudo group
```
id -u zac
adduser zac
passwd zac
usermod -aG wheel zac
chown zac:zac /aemg/moodle

cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
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
```

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

and then start it
```
systemctl start xinetd
```

## Sync code
>In 199, run below command
```
rsync -azpP --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" /aemg/cloudclassroom/moodle zac@192.168.86.26::moodle --delete
```
In desired way, we would like install plugin without any troubles, which will need write permits of the code directory.
```
chown -R apache:apache /aemg/cloudclassroom/moodle
```

>In 199, add this to /home/zac/iwait.sh:
```
rsync -azvp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" --exclude "/aemg/cloudclassroom/moodle/config.php" /aemg/cloudclassroom/moodle zac@192.168.86.26::moodle --delete

# kill exists and restart
ps aux | grep iwait.sh
ps aux | grep inotify

nohup /bin/bash /home/zac/iwait.sh &
```



# Httpd & PHP

## Httpd
``` sh
yum --enablerepo=epel,remi install httpd
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```
## VirtualHost
```
ln -s /aemg/moodle /var/www/moodle
vim /etc/httpd/conf.d/cloudcampus.conf
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

### CentOS 7
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
systemctl start httpd
systemctl enable httpd
systemctl reload httpd

scp zac@192.168.86.65:/etc/php.ini /etc/php.ini.new
mv /etc/php.ini /etc/php.ini.bak
mv /etc/php.ini.new /etc/php.ini
systemctl restart httpd

vim /var/www/html/index.php
# Write and Test
<?php phpinfo(); ?>
```

key points in php.init

```
max_execution_time 300
post_max_size 4096M
upload_max_filesize 4096M
```

### CentOS 8

RHEL 8 provides PHP version 7.2, 7.3 in its official repository


>Command to install the EPEL repository configuration package:
```
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

>Command to install the Remi repository configuration package:
```
dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

>Command to install the yum-utils package (for the yum-config-manager command):
```
dnf install yum-utils
```

>You want a single version which means replacing base packages from the distribution

>Packages have the same name than the base repository, ie php-*

>Some common dependencies are available in remi-safe repository, which is enabled by default

>You have to enable the module stream for 7.2:
```
dnf module reset php
dnf module install php:remi-7.2
```
>Command to upgrade (the repository only provides PHP):
```
dnf update
```

>Command to install additional packages:
```
dnf install php-xxx
```

>Command to install testing packages:
```
dnf --enablerepo=remi-modular-test install php-xxx
```

>Command to check the installed version and available extensions:
```
php --version
php --modules
```

https://rpms.remirepo.net/wizard/


# Add Node to Load Balancer

1. add 192.168.86.26 www4.cloudcampus.com.au to `/etc/hosts`

2. add node to nginx upstream chain
```  
vim /etc/nginx/nginx.conf 
nginx -s reload
```

# Optional

## Monitor
``` sh
scp root@192.168.86.65:/aemg/monitor.sh /aemg/monitor.sh
crontab -e
*/5 * * * * /aemg/monitor.sh > /dev/null 2>&1
```

#### Check and Install Mail Utility
``` bash
which mailx

dnf install postfix
systemctl enable postfix ; systemctl start postfix
dnf install mailx

$ mail root
Subject: quota rise request
Dear admin,
Please increase my disk quota with 1 GB.
Thanks, foo
.
EOT

# Check in root account
mail
```

## Verify

### Verify remote ports if Opened

If Telnet does not come with the system, install it.

``` sh
yum install telnet
# or
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/telnet-0.17-64.el7.x86_64.rpm
rpm -ivh telnet-0.17-64.el7.x86_64.rpm 
```

``` sh
# rsyncd
telnet 192.168.86.199 873
# memcached
telnet 192.168.86.199 22122
# mysql
telnet 192.168.86.200 3306
# maxscale
telnet 192.168.86.200 4006
# httpd
telnet 192.168.86.199 80
lsof -i :80
```
### Verify service

#### Commands
```
systemctl list-unit-files | grep enabled
# or
systemctl list-unit-files --state=enabled

systemctl | grep running
# or
systemctl list-units --type=service --state=running
```

#### Services should be enabled

* chronyd.service 
* crond.service 
* firewalld.service
* mysqld.service **`# for db replication`**
* httpd.service **`# for webserver`**
* sshd.service
* xinetd.service **`# for webserver`**
* nfs-client.target **`# for webserver`**

### Verify firewall

```
firewall-cmd --list-all
```
interfaces: eth0

services: http https ntp rsyncd ssh or mysql