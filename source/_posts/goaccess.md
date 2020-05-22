---
title: goaccess
date: 2020-05-22 11:36:17
categories:
tags:
---
# Intro

GoAccess was designed to be a fast, terminal-based log analyzer. Its core idea is to quickly analyze and view web server statistics in real time without needing to use your browser (great if you want to do a quick analysis of your access log via SSH, or if you simply love working in the terminal).

[goaccess](https://goaccess.io/)
<!--more-->

# Nginx

## Log
```                        
222.183.184.204 [22/May/2020:09:01:45 +0800] 200 "GET /theme/yui_combo.php?m/1589419535/core/dock/dock-loader-min.js HTTP/1.1" 768 https://www.cloudcampus.com.au/login/index.php https www.cloudcampus.com.au "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3756.400 QQBrowser/10.5.4043.400" [192.168.86.199:80]
222.183.184.204 [22/May/2020:09:01:46 +0800] 200 "GET /lib/requirejs.php/1589419535/core/first.js HTTP/1.1" 172531 https://www.cloudcampus.com.au/login/index.php https www.cloudcampus.com.au "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3756.400 QQBrowser/10.5.4043.400" [192.168.86.203:8080]
```
## log_format
```
log_format  main  '$remote_addr [$time_local] $status '
                        '"$request" $body_bytes_sent '
                        '$http_referer $scheme $http_host '
                        '"$http_user_agent" [$upstream_addr]';
```

# Goaccess 

## Edit configuration

``` bash
vim /etc/goaccess.conf
```

add new formatter

```
time-format %H:%M:%S
date-format %d/%b/%Y
#log-format %h %^ [%d:%t %^] "%r" status:%s %b %R %^ %^ %^ %^ %^ [%u] %^ %^
log-format %h [%d:%t %^] %s "%r" %b %R %^ %^ "%u" [%^]
```

## Test
```
goaccess -a -d -f /var/log/nginx/a0522.log -p /etc/goaccess.conf -o /aemg/cloudclassroom/analytics/access_log/report2020-05-21.html

goaccess -a -f /var/log/nginx/access.log > /aemg/cloudclassroom/analytics/access_log/report2020-05-21.html
```

# Cron Job

## Create a bash script
``` bash
# /bin/bash
BIN_DIR="/usr/bin"
DATE="`date --date='yesterday' +%F`"
REPORT="/aemg/cloudclassroom/analytics/access_log/report$DATE.html"
$BIN_DIR/goaccess -f /var/log/nginx/access.log -a > $REPORT
```

## Add the script to crond
``` bash
crontab -e
55 2 * * * /aemg/backups/nginx_report.sh  > /dev/null 2>&1
```