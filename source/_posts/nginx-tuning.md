---
title: Nginx Tuning
date: 2020-01-22 11:59:58
categories:
tags:
---
# Check System Resources

By tuning a software, the first thing is to figure out how many resources are there available and then check if system have limitation to restrict you from using them.

CPUs provides main resources for how many processes can be running simultaneously, it means a lot for web server as http connections really depend on it.

> **`CPUs = Threads per core * cores per socket * sockets`**

Two commands to check:
* nproc - print the number of processing units available
* lscpu - display information about the CPU architecture
* ulimit - get and set user limits

<!--more-->

``` bash
nproc --all
2

lscpu | grep -E '^Thread|^Core|^Socket|^CPU\('
CPU(s):                32
Thread(s) per core:    2
Core(s) per socket:    8
Socket(s):             2

# List System Limitation
ulimit -a

#To change the nofile to 94000 you can do:
ulimit -n 94000
```

# Handle System Limitation

**`/etc/security/limits.conf`** lists the limitation of the system. By running Nginx config test command, we got a warning said the config out the limitation of system permitted.
```
service nginx configtest
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: [warn] 65535 worker_connections exceed open file resource limit: 1024
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Edit and add the following lines.
```
* hard nofile 94000
* soft nofile 94000
* hard nproc 64000
* soft nproc 64000
```

## Do changes in /etc/security/limits.conf require a reboot?
No, but we should close all active sessions windows. They still remember the old values. In other words, log out and back in. Every remote new session or a local secure shell take effect of the limits changes.

# Tuning

## Worker Processes

* **`worker_processes`**
The number of NGINX worker processes (the default is 1). In most cases, running one worker process per CPU core works well, and we recommend setting this directive to auto to achieve that.

* **`worker_connections`**
The maximum number of connections that each worker process can handle simultaneously. The default is 512, but most systems have enough resources to support a larger number.

```
worker_processes  24;
events {
    worker_connections  65535;
}
```

## Access Logging
```
log_format  main  '$remote_addr $remote_user [$time_local] '
                  '$request status:$status $body_bytes_sent '
                  '$http_referer $remote_addr $proxy_add_x_forwarded_for '
                  '$scheme proxy_host:$proxy_host http_host:$http_host host:$host [$http_user_agent] [$upstream_addr] [$request_body]';

access_log  /var/log/nginx/access.log  main buffer=32k flush=5s;
```

## Useful sites

https://www.nginx.com/blog/tuning-nginx/