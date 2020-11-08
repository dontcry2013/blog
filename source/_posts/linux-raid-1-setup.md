---
title: Linux RAID 1 Setup
date: 2020-11-09 10:14:25
categories: [Linux]
tags: [Basic]
---
# RAID 1

RAID stands for Redundant Array of Inexpensive Disks, There are many RAID levels such as RAID 0, RAID 1, RAID 5, RAID 10 etc. we will set up software RAID 1 on a running Linux distribution.

<!--more-->

## Format Hard Drive

``` bash
fdisk /dev/sdb
```

## Create RAID 1 Logical Drive

``` bash
mdadm --examine /dev/sdb /dev/sdc

mdadm --create /dev/md0 --level=mirror --raid-devices=2 /dev/sda6 /dev/sdb2
cat /proc/mdstat

mkfs.ext4 /dev/md0
mkdir /aemg
mount /dev/md0 /aemg
df -h /aemg
/dev/md0   /aemg   ext4   defaults   0   0

mdadm --examine /dev/sda6 /dev/sdb2
mdadm --detail /dev/md0

/dev/sdc1   /media/hdd   ext4   defaults   0   0
/dev/sdc2   /media/hdd2   ext4   defaults   0   0
```

[Tutorial](https://www.linuxbabe.com/linux-server/linux-software-raid-1-setup)