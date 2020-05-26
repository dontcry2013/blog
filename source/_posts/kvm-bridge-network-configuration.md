---
title: KVM Bridge Network Configuration
date: 2020-05-22 21:57:51
categories:
tags:
---
# Host Configuration
## 1.
```bash
vim /etc/sysconfig/network-scripts/ifcfg-br0
```
``` yml
TYPE=Bridge
NAME=br0
DEVICE=br0
DELAY=0
ONBOOT=yes
DEFROUTE=yes
#BOOTPROTO=dhcp
BOOTPROTO=static
IPADDR=192.168.1.112
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=192.168.1.1
```
<!--more-->
## 2.
```bash
vim /etc/sysconfig/network-scripts/ifcfg-enp4s0
```
``` yml
HWADDR=04:D9:F5:D3:59:DB
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME="WiredConn"
UUID=a26cf5d7-ae6e-4945-bdcb-9688d87d1724
ONBOOT=yes
NM_CONTROLLED=no
BRIDGE=br0
BOOTPROTO=none
```
## 3. Restart Network

```bash
systemctl list-units
systemctl restart network.service
```

## 4. Verify
```bash
brctl show

bridge name	bridge id		STP enabled	interfaces
br0		8000.04d9f5d359db	no		enp4s0
```

# KVM

```bash
virsh list --all

 Id    Name                           State
----------------------------------------------------
 6     centos7.0                      running
```
```bash
virsh edit centos7.0
```
```xml
<interface type='bridge'>
  <mac address='...'/>
  <source bridge='br0'/>
  <model type='rtl8139'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```
```
virsh reboot centos7.0
```

# Virsh

```bash
virsh help
mkdir /data/kvmback
virsh dumpxml centos7.0 >/data/kvmback/cent7_back.xml
virsh reboot centos7.0
virsh autostart centos7.0 
virsh autostart --disable snale
```

# Bring up a wireless network

```
ip addr show
3: wlp3s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN

ip link set dev wlp3s0 up
RTNETLINK answers: Operation not possible due to RF-kill

rfkill list all

rfkill unblock all

dmesg | grep phy0

ip addr add 192.168.1.111/24 dev wlp3s0
```

# Route table configuration
```
# show route
netstat -anr
# or
route -n
# or
ip route list
```
```
route add -net 192.168.2.0 netmask 255.255.255.0 dev enp6s0	
# or
ip route add 192.168.2.0/24 dev enp6s0

route add default gw 192.168.2.254	
# or
ip route add default via 192.168.2.254



route delete default gw 114.112.104.1 eth3
route delete default gw 192.168.86.1 eth0
route add default gw 192.168.86.68 dev eth0

route del -net 114.112.104.0 gw 0.0.0.0 netmask 255.255.255.0 dev eth1
```
```
systemctl list-units
systemctl restart NetworkManager.service
```
# Restart eth0
1.
```
systemctl restart network.service
```
2.
```
ifdown eth0;ifup eth0
# or
ip link set dev eth0 down;ip link set dev eth0 up
```


