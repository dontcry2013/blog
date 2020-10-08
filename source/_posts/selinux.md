---
title: SELinux
date: 2020-02-11 15:23:21
categories: [Linux]
tags: [SELinux, CentOS]
---
# Introduction
SELinux is a security mechanism built into the Linux kernel. Linux distributions such as CentOS, RHEL, and Fedora are equipped with SELinux by default.

SELinux improves server security by restricting and defining how a server processes requests and users interact with sockets, network ports, and essential directories.

For example, if an unauthorized user gains access, server access is restricted to a specified section, limiting the damage caused by the breach. SELinux can also obstruct the installation of software packages or terminate processes during regular use.

<!--more-->

```
sestatus

SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```

> vim /etc/selinux/config

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

# Change SELinux Mode

To change the mode from enforcing to permissive type:

`sudo setenforce 0`

To turn the enforcing mode back on, enter:

`sudo setenforce 1`

# Disable SELinux
Open the /etc/selinux/config file and set the SELINUX mod to disabled:
```
sudo shutdown -r now
```


