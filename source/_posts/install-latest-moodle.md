---
title: install-latest-moodle
date: 2020-04-09 23:03:35
categories:
tags:
---
# Install PHP 7.3
``` shell
yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm 
yum install epel-release yum-utils
yum repolist
yum-config-manager --disable remi-php54
yum-config-manager --enable remi-php73
yum install php php-cli php-fpm php-mysqlnd php-zip php-devel php-gd php-mcrypt php-mbstring php-curl php-xml php-pear php-bcmath php-json
yum install php-xmlrpc php-intl php-soap
rpm -qi php-mysqlnd
php -v
php -i | grep php.ini
php -m
```
<!--more-->
# Install the Latest Apache
``` shell
yum info httpd
ls /etc/repo.d
ls /etc/yum.repos.d
cd /etc/yum.repos.d && wget https://repo.codeit.guru/codeit.el`rpm -q --qf "%{VERSION}" $(rpm -q --whatprovides redhat-release)`.repo
yum info httpd
yum install httpd
systemctl start httpd
curl localhost
systemctl enable httpd
firewall-cmd --list-all
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

# Useful Tips

``` shell
# 80 to 8080
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080
# 8080 to 192.168.1.113:8080
firewall-cmd --permanent --add-forward-port=port=8080:proto=tcp:toaddr=192.168.1.113:toport=80
firewall-cmd --permanent --remove-forward-port=port=8080:proto=tcp:toport=80:toaddr=192.168.1.113
```
## masquerade to Allow forward IP
```
firewall-cmd --query-masquerade
firewall-cmd --add-masquerade --permanent
firewall-cmd --remove-masquerade
```