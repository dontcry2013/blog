---
title: Apache Configuration of Default Website
date: 2020-01-17 12:24:32
categories:
tags:
---
# What I Want

To get a direct access to the website, instead of the apache welcome page when access to a IP address.

To change this :

![default](/blog/img/apache-default.png)
<!--more-->
into this: 

![page](/blog/img/moodle-index.png)

# Ubuntu

In Ubuntu, the apache server is changed and different from the original one.

`vim /etc/apache2/apache2.conf`

``` sh
# Include of directories ignores editors' and dpkg's backup files,
# see README.Debian for details.

# Include generic snippets of statements
IncludeOptional conf-enabled/*.conf

# Include the virtual host configurations:
IncludeOptional sites-enabled/*.conf

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
ServerName 192.168.1.105
```
## ServerName

The **`ServerName`**  directive sets the request scheme, hostname and port that the server uses to identify itself. In the context of virtual hosts, the ServerName specifies what hostname must appear in the request's Host: header to match this virtual host. For the default virtual host (this file) this value is not decisive as it is used as a last resort host regardless. However, you must set it for any further virtual host explicitly.

Domain name matches IP, to find your server, but the serverName will match the website(sub path in you server) domain name.

## Website Configuration

``` php
if($_SERVER['HTTP_HOST'] == "192.168.1.105"){
  $CFG->wwwroot   = 'http://192.168.1.105';
  $CFG->dbhost    = 'localhost';
}
```
## Default Configuration
The default config file - **`000-default.conf `** allows every sub folder of `/var/www/http` can be accessed by using url like `IP/{folder-name}`.

``` sh
  <Directory /var/www/html>
          Options Indexes FollowSymLinks MultiViews
          AllowOverride All
          Require all granted
  </Directory>
  
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html
```
