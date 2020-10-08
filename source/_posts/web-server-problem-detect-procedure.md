---
title: Web Server Problem Detect Procedure
date: 2020-03-17 13:36:07
categories: [Solutions, Linux]
tags: [Firewall, SELinux, CentOS]
---
When a new server is setup or when it restarted, we encounter the problems like can not access the website every now and then. Mostly, we should just check firewall and selinux. 

<!--more-->


# Firewall
For webserver, mysql, httpd, ssh are mandatory. So, firewall should not block any port of these service. Issue the following command, to check if it is permanently allowed.
```
[root@localhost zac]# firewall-cmd --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: dhcpv6-client http https mysql rsyncd ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

# SELinux

Sometimes SELinux is the potential a pain in the neck, normally I would stop it before rush into a weird problem.
```
sestatus

# Disable
sudo setenforce 0
```

Open the `/etc/selinux/config` file and set the SELinux mod to disabled.

`sudo shutdown -r now` to make it work.
