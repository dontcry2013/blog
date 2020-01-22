---
title: iptables and firewalld
date: 2020-01-22 16:17:58
categories:
tags:
---

# Commands
```
firewall-cmd --permanent --zone=public --add-source=192.168.86.0/24
firewall-cmd --permanent --zone=public --add-source=210.10.220.42/32
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=mysql
firewall-cmd --reload
firewall-cmd --permanent --list-service
# Or
firewall-cmd --list-all
```

<!--more-->




```
iptables --line -vnL # To check Chain IN_public_allow
```