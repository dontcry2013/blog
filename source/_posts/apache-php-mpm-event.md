---
title: Apache enable mpm-event and PHP Config
date: 2020-01-15 14:13:43
categories: [Tools]
tags: [Tuning]
---
# CentOS

First of all, it's better to check the required packages are available in the system by using following commands:

``` sh
rpm -qa | grep {package-name}
# On a CentOS/RHEL version 6.x/7.x and above
yum list installed {PACKAGE_NAME_HERE}
```

And then install dependencies.

``` sh
yum install httpd httpd-tools mod_ssl php-fpm
```
<!--more-->
## mpm.conf
```
vim /etc/httpd/conf.modules.d/00-mpm.conf 
# LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
LoadModule mpm_event_module modules/mod_mpm_event.so
```

## php.conf
``` sh
vim /etc/httpd/conf.d/php.conf

# Tell the PHP interpreter to handle files with a .php extension.

# Proxy declaration
<Proxy "unix:/var/run/php-fpm/default.sock|fcgi://php-fpm">
	# we must declare a parameter in here (doesn't matter which) or it'll not register the proxy ahead of time
    	ProxySet disablereuse=off
</Proxy>

# Redirect to the proxy
<FilesMatch \.php$>
	SetHandler proxy:fcgi://php-fpm
</FilesMatch>

#
# Allow php to handle Multiviews
#
AddType text/html .php

#
# Add index.php to the list of files that will be served as directory
# indexes.
#
DirectoryIndex index.php

#
# Uncomment the following lines to allow PHP to pretty-print .phps
# files as PHP source code:
#
#<FilesMatch \.phps$>
#	SetHandler application/x-httpd-php-source
#</FilesMatch>
```

## php-fpm

``` sh
vim /etc/php-fpm.d/www.conf
; listen = 127.0.0.1:9000
listen = /var/run/php-fpm/default.sock
...
listen.allowed_clients = 127.0.0.1
listen.owner = apache
listen.group = apache
listen.mode = 0660
user = apache
group = apache
```
## Enable and Start Package
```
[root@web01 ~]# systemctl enable php-fpm
[root@web01 ~]# systemctl enable httpd
[root@web01 ~]# systemctl start php-fpm
[root@web01 ~]# systemctl start httpd
[root@web01 ~]# firewall-cmd --zone=public --permanent --add-service=http
[root@web01 ~]# firewall-cmd --zone=public --permanent --add-service=https
[root@web01 ~]# firewall-cmd --reload
```

# Ubuntu

## Package Manager for Debian to Check if Package Installed
``` sh
dpkg -s {package-name}
# or
dpkg-query -l
```

## Debian-like OS Apache

According to the official doc of Apache, the Apache 2 web server configuration in Debian is quite different to
upstream's suggested way to configure the web server. This is because Debian's
default Apache2 installation attempts to make adding and removing modules,
virtual hosts, and extra configuration directives as flexible as possible, in
order to make automating the changes and administering the server as easy as
possible.

> `sudo apachectl -V` will show which mod Apache is using.

In order to use event mode of Apache, we should remove the pre-installed php, because it depends on pre-fork mod.

## 1. Remove or disable the Apache PHP module
Because the Ubuntu packages insist on **`prefork`** Apache when installing PHP, we have to separate them as it's restrict us to change other mod. Use apt to uninstall libapache2-mod-php7.0 if you no longer need the package:

> `sudo apt-get remove libapache2-mod-php7.0`

Alternatively, you might disable the php7.0 Apache module instead, but this will not remove the apt package from your system, which leaves behind annoying system cruft.

> `sudo a2dismod php7.0`

## 2. Switch Apache to event mode and enable fcgid
I believe these commands should do the trick:

```
sudo a2dismod mpm_prefork
sudo a2enmod mpm_event
sudo a2enmod proxy_fcgi
```

## 3. Install PHP-FPM
I already have PHP 7 installed with its various modules, so I just install PHP-FPM with this command:

> `sudo apt-get install php7.0-fpm`

## 4. Edit your VirtualHost configuration to handle PHP files with PHP-FPM:
In my case, I edited the default SSL host, /etc/apache2/sites-available/default-ssl.conf, and added this line right near the top:

ProxyPassMatch ^/(.*\.php(/.*)?)$ unix:/run/php/php7.0-fpm.sock|fcgi://localhost/var/www/html/
IMPORTANT This instructs Apache to handle PHP file requests with PHP-FPRM and the the path in this directive (/run/php/php7.0-fpm.sock) must match the path specified by the listen directive in the file /etc/php/7.0/fpm/pool.d/www.conf

## 5. Restart Apache

> `sudo service apache2 restart`

To check if event mode is enabled, use this command:

> `sudo apachectl -V`

In the output, you should see this:
```
Server MPM:     event
Try creating a phpinfo page and accessing it in your browser. You should see Server API: FPM/FastCGI in the output.
```