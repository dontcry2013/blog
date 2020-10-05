---
title: Create Bootable Flash Stick
date: 2020-10-03 23:07:16
categories:
tags:
---
# Mac OSX 

<!--more-->

```
diskutil list
diskutil umount /dev/disk3s1
dd if=/dev/disk3s1 of=usb.img bs=4096
dd if=usb.img of=/dev/disk2s1 bs=4096
```