---
title: PHP
date: 2016-04-29 18:34:12
categories: Basic
tags: [Apache, PHP, Ubuntu]
---
# Apache
Start Apache: **`sudo apachectl start`**

Stop Apache: **`sudo apachectl stop`**

Restart Apache: **`sudo apachectl restart`**

***
<!--more-->

# Open PHP
To open PHP, need to modify the following config file

```sh
sudo vim /etc/apache2/httpd.conf
```

Find the line **`#LoadModule php5_module libexec/apache2/libphp5.so`**, remove the **`#`** sign.

In mac, the default Apache directory is **`/Library/WebServer/Documents`**, create a file named **`index.php`**, if it's not exist, write **`<?php phpinfo(); ?>`** as the first line. Open browser to access localhost, if the PHP info page displayed, that means PHP is configured to open successfully.

# Change Apache Default Directory

In mac, the directory of **`/Library/WebServer/Documents`** is hidden by default, it is inconvenient to access, so we can change it to someone we'd like to access.

Open terminal and input: **`sudo vim /etc/apache2/httpd.conf`**

Find the following lines
```
DocumentRoot "/Library/WebServer/Documents"
<Directory "/Library/WebServer/Documents">
```
Replace the directory with you preferred one, restart Apache, move the just created **`index.php`** to the directory, and then test in the browser to see if the php info page still working.