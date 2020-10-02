---
title: GoAccess
date: 2020-05-22 11:36:17
categories:
tags: [Nginx, Shell]
---
# Intro

GoAccess was designed to be a fast, terminal-based log analyzer. Its core idea is to quickly analyze and view web server statistics in real time without needing to use your browser (great if you want to do a quick analysis of your access log via SSH, or if you simply love working in the terminal).

[GoAccess](https://goaccess.io/)
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
## SPECIFIERS
* %x A date and time field matching the time-format and date-format variables. This is used when a timestamp is given instead of the date and time being in two separate variables.
* %t time field matching the time-format variable.
* %d date field matching the date-format variable.
* %v The server name according to the canonical name setting (Server Blocks or Virtual Host).
* %e This is the userid of the person requesting the document as determined by HTTP authentication.
* %h host (the client IP address, either IPv4 or IPv6)
* %r The request line from the client. This requires specific delimiters around the request (single quotes, double quotes, etc) to be parsable. Otherwise, use a combination of special format specifiers such as %m, %U, %q and %H to parse individual fields.
  >Note: Use either %r to get the full request OR %m, %U, %q and %H to form your request, do not use both.
* %m The request method.
* %U The URL path requested.
  >Note: If the query string is in %U, there is no need to use %q. However, if the URL path, does not include any query string, you may use %q and the query string will be appended to the request.
* %q The query string.
* %H The request protocol.
* %s The status code that the server sends back to the client.
* %b The size of the object returned to the client.
* %R The "Referer" HTTP request header.
* %u The user-agent HTTP request header.
* %D The time taken to serve the request, in microseconds.
* %T The time taken to serve the request, in seconds with milliseconds resolution.
* %L The time taken to serve the request, in milliseconds as a decimal number.
* %^ Ignore this field.
* %~ Move forward through the log string until a non-space (!isspace) char is found.
* ~h The host (the client IP address, either IPv4 or IPv6) in a X-Forwarded-For (XFF) field.

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