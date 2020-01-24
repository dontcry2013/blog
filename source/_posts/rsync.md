---
title: rsync and xinetd
date: 2020-01-23 11:50:52
categories:
tags:
---

![default](/blog/img/rsync-error.png)
For rsync daemon, with the first glance of the above picture we would think there is no rsync server running. But the fact is not that simple, it could be managed by **`xinetd`**.


A xinetd, the eXtended InterNET Daemon, manages Internet-based connectivity. It offers a more secure extension to or version of inetd, the Internet daemon.

xinetd starts programs that provide Internet services. Instead of having such servers started at system initialization time, and be dormant until a connection request arrives, xinetd is the only daemon process started and it listens on all service ports for the services listed in its configuration file. When a request comes in, xinetd starts the appropriate server. Because of the way it operates, xinetd (as well as inetd) is also referred to as a super-server.

<!--more-->
# xinetd Configuration files location
Following are important configuration files for xinetd:

* /etc/xinetd.conf – The global xinetd configuration file.
* /etc/xinetd.d/ directory – The directory containing all service-specific files such as ftp

# Problem Solve

Run `chkconfig --list` to check if rsync is managed by xinted, and check rsync configuration files location: `/etc/rsyncd/rsyncd.conf`.

```
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsync.log
port = 873
```

**`Attention`**: If rsync is failed to startup, no error displayed in the terminal, the error log only be recorded in log file.

# Commands
```
/usr/bin/rsync --daemon --config /etc/rsyncd/rsyncd.conf

chkconfig --list

ps aux | grep rsync

service --status-all

netstat -anltp

service xinetd status
# Or
etc/init.d/xinetd status
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