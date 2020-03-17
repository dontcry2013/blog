---
title: iptables and firewalld
date: 2020-01-22 16:17:58
categories: [Linux]
tags: [CentOS]
---

# Firewalld

![Firewall stack][Firewall stack]
<!--more-->

–permanent param added to prevent invalid when restart.

# Check Status
Get default services that are available.
```
firewall-cmd --get-services
```
```
firewall-cmd --list-all-zones
firewall-cmd --get-active-zones
```
```
firewall-cmd --list-all
firewall-cmd --permanent --zone=public --list-sources
firewall-cmd --permanent --list-service
firewall-cmd --list-rich-rules
firewall-cmd --permanent --list-ports
```
# Set Zone to Active
```
firewall-cmd --permanent --zone=public --change-interface=eth0
```
# Set Zone to Drop
```
firewall-cmd --permanent --zone=public --set-target=DROP
```
# Commands
make it count.
```
firewall-cmd --reload
```
others
```
firewall-cmd --permanent --zone=public --add-source=192.168.86.0/24
firewall-cmd --permanent --zone=public --add-source=210.10.220.42/32

# Query state of port
firewall-cmd --zone=public --query-port=80/tcp
# Open port
firewall-cmd --permanent --zone=public --add-port=8080/tcp
# Remove
firewall-cmd --permanent --zone=public --remove-port=8080/tcp

firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --add-service=mysql
firewall-cmd --zone=public  --permanent --remove-service=ssh
```

# Rich Rules

[firewalld.richlanguage(5)](https://jpopelka.fedorapeople.org/firewalld/doc/firewalld.richlanguage.html)

```
man firewalld.richlanguage
```

`--add-rich-rule, --list-rich-rules, --remove-rich-rule` to manage.


```
# Allow all traffic from 192.168.0.14 
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address=192.168.0.14 accept'

# Reject all traffic from 192.168.0.14 to port 22
firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="192.168.1.10" port port=22 protocol=tcp reject'

firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="1.2.3.4/32"
  port protocol="tcp" port="4567" accept'

# Or

firewall-cmd --zone=work --add-source=00:11:22:33:44:55
firewall-cmd --zone=work --add-rich-rule='rule source mac=11:22:33:44:55:66 drop'
```

```
firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" reject'

firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" port port="22" protocol="tcp" reject'
```

# iptables

```
iptables --list
iptables -P INPUT DROP
iptables --line -vnL # To check Chain IN_public_allow
/sbin/iptables -A INPUT -m mac --mac-source XX:XX:XX:XX:XX:XX -j DROP
/sbin/iptables -A INPUT -p tcp --destination-port 22 -m mac --mac-source XX:XX:XX:XX:XX:XX -j ACCEPT
```


# Issues

mysql can not connect to ip and port.

Got error: `no route to host`

```
> telnet 192.168.1.108 4006
Trying 192.168.1.108...
telnet: Unable to connect to remote host: No route to host
```
Shutdown Firewalld, still no good.
Issue this works:
```
iptables -F 
```

[Firewall stack]: /blog/img/firewall_stack.png "Firewall stack"


[FirewallD Official Websites](http://www.firewalld.org/)

[RHEL 7 USING FIREWALLS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls#sec-Introduction_to_firewalld)