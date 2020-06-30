---
title: wordpress study log
date: 2020-06-30 22:44:19
categories:
tags:
---
# XAMPP

``` bash
# check default username and password
/Applications/XAMPP/xamppfiles/xampp security
# get right privileges to install & uninstall plugins
chown -R daemon:daemon /Applications/XAMPP/htdocs/wp
```
<!--more-->

# MySQL

For MySQL 8.x, should use mysql_native_password.

``` sql
ALTER USER 'username'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

---