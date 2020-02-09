---
title: rsync and xinetd
date: 2020-01-23 11:50:52
categories: [Tools]
tags: [CentOS]
---

![default](/blog/img/rsync-error.png)
For rsync daemon, with the first glance of the above picture we would think there is no rsync server running. But the fact is not that simple, it could be managed by **`xinetd`**.

A xinetd, the eXtended InterNET Daemon, manages Internet-based connectivity. It offers a more secure extension to or version of inetd, the Internet daemon.

xinetd starts programs that provide Internet services. Instead of having such servers started at system initialization time, and be dormant until a connection request arrives, xinetd is the only daemon process started and it listens on all service ports for the services listed in its configuration file. When a request comes in, xinetd starts the appropriate server. Because of the way it operates, xinetd (as well as inetd) is also referred to as a super-server.

<!--more-->
# rsync Configuration

### Server Configuration

```
[moodleData]
path = /aemg/moodledata/filedir
list=no
auth users = zac
```

### Client 

```
rsync -azp --password-file="/home/zac/pass" --log-file="/var/log/rsync" /aemg/moodledata/filedir/ zac@192.168.86.200::moodleData --delete
```

**`/aemg/moodledata/filedir/`** should be exactly like this. This means to push all files under `filedir` to the server. Without **`/`**, rsync will create another filedir under **`/aemg/moodledata/filedir`**.


# Configuration Files' Location of xinetd
Following are important configuration files for xinetd:

* /etc/xinetd.conf – The global xinetd configuration file.
* /etc/xinetd.d/ directory – The directory containing all service-specific files such as ftp and rsync

# Problem Solve

Run `chkconfig --list` to check if rsync is managed by xinted, and check rsync configuration files location: `/etc/rsyncd/rsyncd.conf`.

```
address = 192.168.1.100
pid file = /var/run/rsyncd.pid
log file = /var/log/rsync.log
port = 873
```
## Explanation of Configuration

* pid file - process id file of the daemon
* port - port will be listening by the daemon, can be ignored when use xinetd
* address - address on which the rsync daemon listens, can be ignored when use xinetd

**`Attention`**

> Sometime IP address could be an issue when you got internal and external addresses, if connections come from external use external address, internal works like the same.

> If rsync is failed to startup, no error log might be displayed on the terminal, the error log could  be recorded in log file.

## Password File Permission
Another thing should be noticed is: in Linux, every secret file should be only readable by the user itself, so change the file permissions after create one.
```
chmod 600 rsyncd.secrets
```

# Check auto-start Programs on Boot
```
# CentOS 6
chkconfig –-list xinetd

# CentOS 7
systemctl list-unit-files | grep xinetd

# Restart xinetd
service xinetd restart
```


# Commands

Both CentOS 6 and 7 can use **`service`** command

```
service xinetd status
service rsyncd restart
service --status-all

# Check All Listening Ports
netstat -anltp

# CentOS 6 Check xinetd Status
/etc/init.d/xinetd status
```

Edit **`/etc/xinetd.d/rsync`**

```
service rsync
{
        disable = no
        flags           = IPv6
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/bin/rsync
        server_args     = --daemon   --config=/etc/rsyncd/rsyncd.conf
        log_on_failure  += USERID
}
```


# Conclusion

xinetd is a really powerful server running in a lot of Linux versions. It is can be used to secure the server, protect the server from **`Denial of Services`** attack.