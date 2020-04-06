---
title: phpmyadmin
date: 2020-04-06 13:33:17
categories:
tags:
---
# Install the Latest Version

If you're using `yum` to install software in CentOS, most likely it is not the latest version, just like install phpmyadmin, run `yum install phpmyadmin`, it will only install version 4.4, which the latest version is 5.02 at the time I'm writing this article.

<!--more-->

```
[root@mel-ryzen conf.d]# yum install phpmyadmin
Loaded plugins: changelog, fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: ftp.swin.edu.au
 * epel: mirror.aarnet.edu.au
 * extras: mirror.colocity.com
 * remi-php72: remi.conetix.com.au
 * remi-safe: remi.conetix.com.au
 * updates: mirror.colocity.com
Resolving Dependencies
--> Running transaction check
---> Package phpMyAdmin.noarch 0:4.4.15.10-4.el7 will be installed
--> Processing Dependency: php-php-gettext for package: phpMyAdmin-4.4.15.10-4.el7.noarch
--> Processing Dependency: php-tcpdf for package: phpMyAdmin-4.4.15.10-4.el7.noarch
--> Processing Dependency: php-tcpdf-dejavu-sans-fonts for package: phpMyAdmin-4.4.15.10-4.el7.noarch
--> Running transaction check
---> Package php-php-gettext.noarch 0:1.0.12-1.el7 will be installed
---> Package php-tcpdf.noarch 0:6.2.26-1.el7 will be installed
--> Processing Dependency: php-bcmath for package: php-tcpdf-6.2.26-1.el7.noarch
--> Processing Dependency: php-composer(fedora/autoloader) for package: php-tcpdf-6.2.26-1.el7.noarch
--> Processing Dependency: php-posix for package: php-tcpdf-6.2.26-1.el7.noarch
--> Processing Dependency: php-tidy for package: php-tcpdf-6.2.26-1.el7.noarch
---> Package php-tcpdf-dejavu-sans-fonts.noarch 0:6.2.26-1.el7 will be installed
--> Running transaction check
---> Package php-bcmath.x86_64 0:7.2.29-1.el7.remi will be installed
---> Package php-fedora-autoloader.noarch 0:1.0.1-2.el7 will be installed
---> Package php-process.x86_64 0:7.2.29-1.el7.remi will be installed
---> Package php-tidy.x86_64 0:7.2.29-1.el7.remi will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================================================================================================
 Package                                                Arch                              Version                                      Repository                             Size
===================================================================================================================================================================================
Installing:
 phpMyAdmin                                             noarch                            4.4.15.10-4.el7                              epel                                  4.7 M
Installing for dependencies:
 php-bcmath                                             x86_64                            7.2.29-1.el7.remi                            remi-php72                             74 k
 php-fedora-autoloader                                  noarch                            1.0.1-2.el7                                  epel                                   11 k
 php-php-gettext                                        noarch                            1.0.12-1.el7                                 epel                                   23 k
 php-process                                            x86_64                            7.2.29-1.el7.remi                            remi-php72                             84 k
 php-tcpdf                                              noarch                            6.2.26-1.el7                                 epel                                  2.1 M
 php-tcpdf-dejavu-sans-fonts                            noarch                            6.2.26-1.el7                                 epel                                  257 k
 php-tidy                                               x86_64                            7.2.29-1.el7.remi                            remi-php72                             66 k

Transaction Summary
===================================================================================================================================================================================
```

To solve this, we should enable remi repo by using the following command:

```
yum --enablerepo=remi install phpmyadmin

Dependencies Resolved

===================================================================================================================================================================================
 Package                                                      Arch                           Version                                      Repository                          Size
===================================================================================================================================================================================
Installing:
 phpMyAdmin                                                   noarch                         5.0.2-2.el7.remi                             remi                               6.9 M
Installing for dependencies:
 composer                                                     noarch                         1.10.1-1.el7.remi                            remi                               407 k
```

# Difference between apache 2.2 and 2.4

Different apache version could be another the pain in the neck, by using version 2.4, we should allow foreign ip to access our phpmyadmin server, so edit the configuration file, and add the ip address in `<Directory /usr/share/phpMyAdmin/>`

```
[root@mel-ryzen Downloads]# vim /etc/httpd/conf.d/phpMyAdmin.conf 

# phpMyAdmin - Web based MySQL browser written in php
# 
# Allows only localhost by default
#
# But allowing phpMyAdmin to anyone other than localhost should be considered
# dangerous unless properly secured by SSL

Alias /phpMyAdmin /usr/share/phpMyAdmin
Alias /phpmyadmin /usr/share/phpMyAdmin

<Directory /usr/share/phpMyAdmin/>
   AddDefaultCharset UTF-8

   Require local
   Require ip 192.168.1.112
   Require ip 192.168.1.107
   Require all granted
</Directory>

<Directory /usr/share/phpMyAdmin/setup/>
   Require local
   #Require ip 192.168.1.107
   #Require all granted
</Directory>

# These directories do not require access over HTTP - taken from the original
# phpMyAdmin upstream tarball
#
<Directory /usr/share/phpMyAdmin/libraries/>
    Require all denied
</Directory>

<Directory /usr/share/phpMyAdmin/templates/>
    Require all denied
</Directory>

<Directory /usr/share/phpMyAdmin/setup/lib/>
    Require all denied
</Directory>

<Directory /usr/share/phpMyAdmin/setup/frames/>
    Require all denied
</Directory>

# This configuration prevents mod_security at phpMyAdmin directories from
# filtering SQL etc.  This may break your mod_security implementation.
#
#<IfModule mod_security.c>
#    <Directory /usr/share/phpMyAdmin/>
#        SecRuleInheritance Off
#    </Directory>
#</IfModule>
```
# Official doc
The [phpMyAdmin official doc](https://docs.phpmyadmin.net/en/latest/) would be very useful, as it gives instruction of installation for latest version, and also use different ways, for example, the very popular docker.