---
title: iptables and firewalld
date: 2020-01-22 16:17:58
categories:
tags:
---
# 

<!--more-->


```
firewall-cmd --permanent --zone=public --add-source=192.168.86.0/24
firewall-cmd --permanent --zone=public --add-source=210.10.220.42/32
firewall-cmd --add-service=http --permanent
firewall-cmd --reload
firewall-cmd --list-all
```