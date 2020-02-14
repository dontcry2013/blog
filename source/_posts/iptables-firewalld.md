---
title: iptables and firewalld
date: 2020-01-22 16:17:58
categories: [Linux]
tags: [CentOS]
---
![Firewall stack][Firewall stack]
<!--more-->

–permanent param added to prevent invalid when restart.

# Commands
```
firewall-cmd --permanent --zone=public --list-sources
firewall-cmd --permanent --zone=public --add-source=192.168.86.0/24
firewall-cmd --permanent --zone=public --add-source=210.10.220.42/32

# Query state of port
firewall-cmd --zone=public --query-port=80/tcp
# Open port
firewall-cmd --permanent --zone=public --add-port=8080/tcp
# Remove
firewall-cmd --permanent --zone=public --remove-port=8080/tcp
firewall-cmd --permanent --list-ports

firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=mysql
firewall-cmd --reload
firewall-cmd --permanent --list-service
firewall-cmd --list-all-zones
firewall-cmd --list-rich-rules

# Allow all traffic from 192.168.0.14 
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address=192.168.0.14 accept'
# Reject all traffic from 192.168.0.14 to port 22
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="192.168.1.10" port port=22 protocol=tcp reject'

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