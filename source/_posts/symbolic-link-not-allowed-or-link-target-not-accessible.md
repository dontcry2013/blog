---
title: Symbolic link not allowed or link target not accessible
date: 2020-04-10 19:14:55
categories:
tags:
---
# Problem Description

apache2 can not access the symbolic link which located in default directory. Gives error "Forbidden You don't have permission to access ... on this server".

<!--more-->


# Detection

## Check the Error Log

* Apache error log is in `/var/log/httpd`, the error is `"AH00037: Symbolic link not allowed or link target not accessible: /var/www/moodle"`.
  
## Do not Use Symbolic Link

* Create a directory which not use symbolic link, like `/var/www/test`, it turns out OK.

# Solution

chmod o+x /root/

The fix is immediate - no need to restart apache.

# Reason

```
[root@localhost /]# ll
total 31
lrwxrwxrwx.   1 root root     7 Nov 25  2018 bin -> usr/bin
dr-xr-xr-x.   6 root root  4096 Mar 24 11:58 boot
drwxr-xr-x    2 root root   258 Apr  2 12:44 com.fr.intelli.record.FocusPoint
drwxr-xr-x   21 root root  3520 Apr  7 11:02 dev
drwxr-xr-x. 157 root root 12288 Apr 10 00:24 etc
drwxr-xr-x.   6 root root    58 Apr  9 13:50 home
lrwxrwxrwx.   1 root root     7 Nov 25  2018 lib -> usr/lib
lrwxrwxrwx.   1 root root     9 Nov 25  2018 lib64 -> usr/lib64
drwx------.   2 root root     6 Nov 25  2018 lost+found
drwxr-xr-x.   2 root root     6 Apr 11  2018 media
drwxr-xr-x.   2 root root     6 Apr 11  2018 mnt
drwxr-xr-x.   6 root root    92 Apr  9 13:26 opt
dr-xr-xr-x  518 root root     0 Apr  7 11:02 proc
dr-xr-x---.  13 root root  4096 Apr 10 18:51 root
drwxr-xr-x   44 root root  1360 Apr 10 06:25 run
lrwxrwxrwx.   1 root root     8 Nov 25  2018 sbin -> usr/sbin
drwxr-xr-x.   2 root root     6 Apr 11  2018 srv
dr-xr-xr-x   13 root root     0 Apr  7 11:02 sys
drwxrwxrwt   24 root root   600 Apr 10 03:07 tmp
drwxr-xr-x.  13 root root   155 Apr  9 13:16 usr
drwxr-xr-x.  22 root root  4096 Mar 27 16:54 var
```

* Symbolic link needs `X` right for every directory in the source link, `/root` is the culprit.
* For others, they don't have execution right on `root` direction