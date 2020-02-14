---
title: iptables and firewalld
date: 2020-01-22 16:17:58
categories: [Linux]
tags: [CentOS]
---
![Firewall stack][Firewall stack]
<!--more-->

# Commands
```
firewall-cmd --permanent --zone=public --list-sources
firewall-cmd --permanent --zone=public --add-source=192.168.86.0/24
firewall-cmd --permanent --zone=public --add-source=210.10.220.42/32
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=mysql
firewall-cmd --reload
firewall-cmd --permanent --list-service
firewall-cmd --permanent --list-ports
# Or
firewall-cmd --list-all

firewall-cmd --zone=work --add-source=00:11:22:33:44:55
firewall-cmd --zone=work --add-rich-rule='rule source mac=11:22:33:44:55:66 drop'
```





```
iptables --list
iptables -P INPUT DROP
iptables --line -vnL # To check Chain IN_public_allow
/sbin/iptables -A INPUT -m mac --mac-source XX:XX:XX:XX:XX:XX -j DROP
/sbin/iptables -A INPUT -p tcp --destination-port 22 -m mac --mac-source XX:XX:XX:XX:XX:XX -j ACCEPT
```


[Firewall stack]: /blog/img/firewall_stack.png "Firewall stack"