---
title: nginx access log analysis command
date: 2020-02-20 10:31:14
categories:
tags:
---
# Procedure
* First of All, Get the Target

```
gunzip /var/log/nginx/access.log-20200219.gz
```
<!--more-->
* Take a Sample

```
head -n 10 access.log-20200220
```

```
120.150.106.31 - [19/Feb/2020:04:23:13 +0800] GET /theme/image.php/lambda/theme/1580192308/bg/block-divider HTTP/1.1 status:200 118 https://www.cloudcampus.com.au/theme/styles.php/lambda/1580192318_1/all 120.150.106.31 120.150.106.31 https proxy_host:myapp1 http_host:www.cloudcampus.com.au [Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36] [-] [-]

120.150.106.31 - [19/Feb/2020:04:23:15 +0800] GET /course/view.php?id=3163 HTTP/1.1 status:200 290827 https://www.cloudcampus.com.au/my/ 120.150.106.31 120.150.106.31 https proxy_host:myapp1 http_host:www.cloudcampus.com.au [Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36] [114.112.104.204:80] [-]

...
```

* Compare with the Log Format

```
log_format  main  '$remote_addr $remote_user [$time_local] '
'$request status:$status $body_bytes_sent '
'$http_referer $remote_addr $proxy_add_x_forwarded_for '
'$scheme proxy_host:$proxy_host http_host:$http_host '
'[$http_user_agent] [$upstream_addr] [$request_body]';
```

# Command List
```
history | tail -n60 

# Get $request and $upstream_addr
cat access.log-20200219 | awk '{print $6  $(NF-1)}' > test.log

# Get $request From None-Memory-Cache
cat test.log | grep -v [-] | awk -F ' ' '{print $1}' | sort | uniq -c | sort -n -r

# Get $upstream_addr
cat access.log-20200219 | awk -F '[' '{print $4}' | awk '{print substr($1, 1, length($1)-1)}' | sort > test.log
```

* `>>` used to append to the file
* `>` will overwrite

