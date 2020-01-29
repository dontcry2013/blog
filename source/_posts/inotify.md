---
title: inotify
date: 2020-01-24 10:18:39
categories:
tags:
---
# 

<!--more-->


# Install 

## Install from Source
``` bash
# Check core version, must latter than 2.6.13
uname -r

rpm -qa | grep gcc
rpm -qa | grep make
# Or
yum list installed gcc 
yum list installed make
# Or
yum install gcc make

wget -c https://github.com/rvoicilas/inotify-tools/archive/3.20.1.tar.gz

tar xvzf 3.20.1.tar.gz 

./configure && make && make install
```
## Install from Repo

```
yum install -y inotify-tools
```
# inotifywait Example
```
inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e create

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e delete

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e close_write

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format "%T %w%f EVENT: %e" -e move

inotifywait -mrq  /tmp/test --timefmt "%d-%m-%y %H:%M" --format '[%T]%w%f[%e]' -e create,close_write,delete,move
```

# Complete Shell Script
```
#!/bin/bash
Path=/home/zac/inotify-test

/usr/bin/inotifywait -mrq --timefmt "%d-%m-%y %H:%M" --format '[%T]%w%f[%e]' -e create,close_write,delete,move $Path  | while read -r line
do
	if [ "$line" ];then
    	echo "${line}"
      rsync -avzp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" /aemg/cloudclassroom/moodle zac@192.168.1.6::moodle --delete
        rsync -azvp --password-file="/home/zac/rsync_pass" --log-file="/var/log/rsync" /aemg/cloudclassroom/moodle zac@192.168.1.23::moodle --delete
	fi
done


```
# Useful sites
## GitHub repo wiki of inotify-tools
https://github.com/inotify-tools/inotify-tools/wiki

## inofitywait Introduction
https://linux.die.net/man/1/inotifywait

## A Tool for Syntax Check of Shell
https://www.shellcheck.net/