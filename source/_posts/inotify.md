---
title: inotify
date: 2020-01-24 10:18:39
categories:
tags:
---
# Real-time Data Synchronization
In production, there is a need to back up data to a backup server when the file data in the directory changes. The realization of such requirements requires the following two points:

* Use the monitoring service (inotify) to monitor changes to the directory files of the server to be synchronized
When the directory data is changed.
* Use the rsync service to send the data to the backup server. Therefore, real-time data synchronization can be achieved by using rsync + inotify.

<!--more-->
# Install 

## Install from Source
``` bash
# Check core version, must latter than 2.6.13
uname -r

# Check and Install the Compile Tools
rpm -qa | grep gcc
rpm -qa | grep make
# Or
yum list installed gcc 
yum list installed make
# Or
yum install gcc make

# Get Source Code and Install
wget -c https://github.com/rvoicilas/inotify-tools/archive/3.20.1.tar.gz

tar xvzf 3.20.1.tar.gz 

./configure && make && make install
```
## Install from Repo(More Convenient Way)

```
yum install -y inotify-tools
```

# Example and Explanation of inotifywait

## Example
```
inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e create

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e delete

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e close_write

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e move

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format '[%T]%w%f[%e]' -e create,close_write,delete,move
```

## Explanation

> -m, --monitor

Instead of exiting after receiving a single event, execute indefinitely. The default behaviour is to exit after the first event occurs.

> -d, --daemon

Same as --monitor, except run in the background logging events to a file that must be specified by --outfile. Implies --syslog.

> -r, --recursive

Watch all subdirectories of any directories passed as arguments. Watches will be set up recursively to an unlimited depth. Symbolic links are not traversed. Newly created subdirectories will also be watched.

>-q, --quiet

If specified once, the program will be less verbose. Specifically, it will not state when it has completed establishing all inotify watches.

> --format \<fmt\>

Output in a user-specified format, using printf-like syntax. The event strings output are limited to around 4000 characters and will be truncated to this length. The following conversions are supported:

%w
This will be replaced with the name of the Watched file on which an event occurred.

%f
When an event occurs within a directory, this will be replaced with the name of the File which caused the event to occur. Otherwise, this will be replaced with an empty string.

%e
Replaced with the Event(s) which occurred, comma-separated.

%Xe
Replaced with the Event(s) which occurred, separated by whichever character is in the place of 'X'.

%T
Replaced with the current Time in the format specified by the --timefmt option, which should be a format string suitable for passing to strftime(3).

# Complete Shell Script
``` bash
#!/bin/bash
Path=/home/zac/inotify-test

/usr/bin/inotifywait -mrq --timefmt "%d-%m-%y %H:%M" --format '[%T]%w%f[%e]' -e create,close_write,delete,move $Path  | while read -r line
do
	if [ "$line" ];then
		echo "${line}"
		rsync -avzp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" --exclude "/code/config.php" /aemg/cloudclassroom/moodle zac@192.168.1.6::moodle --delete
		rsync -azvp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" --exclude "/code/config.php" /aemg/cloudclassroom/moodle zac@192.168.1.23::moodle --delete
	fi
done
```
# Useful Sites
## GitHub Repo wiki of inotify-tools
https://github.com/inotify-tools/inotify-tools/wiki

## inofitywait Introduction
https://linux.die.net/man/1/inotifywait

## A Tool for Syntax Check of Shell
https://www.shellcheck.net/