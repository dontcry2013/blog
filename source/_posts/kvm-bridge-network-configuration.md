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


# Centos 8

```
nmcli conn add type bridge con-name br0 ifname br0

# edit /etc/sysconfig/network-scripts files

nmcli conn up br0

nmcli conn down enp4s0

nmcli conn up enp4s0

nmcli conn show  --active
NAME     UUID                                  TYPE      DEVICE  
br0      ec328a66-4eb0-4e3c-b4fd-e968378b9121  bridge    br0     
docker0  97e90f37-0474-4932-b8ef-a9f2fb22c090  bridge    docker0 
virbr0   8403786d-26b0-4e13-b28b-90c9c5825475  bridge    virbr0  
enp4s0   10ddb735-cb66-43be-8865-0c2cec138e45  ethernet  enp4s0  

bridge link show
2: enp4s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br0 state forwarding priority 32 cost 100 
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 master virbr0 state disabled priority 32 cost 100 
12: vethe5fea27@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master docker0 state forwarding priority 32 cost 2 
```

Centos 8 host should enable masquerade forwarding, to prevent dns problem, as you may able to ping ip, but cannot ping url.

```
firewall-cmd --permanent --zone=public --add-masquerade
```

# Centos 7 NAT Gateway
```
[root@Firewall network-scripts]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:c7:22:d2 brd ff:ff:ff:ff:ff:ff
    inet 114.112.104.202/24 brd 114.112.104.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fec7:22d2/64 scope link 
       valid_lft forever preferred_lft forever
4: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:1c:87:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.86.1/24 brd 192.168.86.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe1c:870c/64 scope link 
       valid_lft forever preferred_lft forever

# eth1 is internal, eth0 is external
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s 192.168.86.0/24
firewall-cmd --permanent --direct --passthrough ipv4 -I FORWARD -i eth1 -j ACCEPT
firewall-cmd --permanent --direct --get-all-passthroughs
firewall-cmd --permanent --direct --get-passthroughs ipv4

systemctl status iptables
```

