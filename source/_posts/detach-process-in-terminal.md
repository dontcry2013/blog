---
title: Start Linux Command in Background and Detach Process in Terminal
date: 2020-02-03 15:37:17
categories: [Linux]
tags:
---
# How to Start a Linux Process or Command in Background

If a process is already in execution, such as the tar command example below, simply press **`Ctrl+Z`** to stop it then enter the command bg to continue with its execution in the background as a job.

You can view all your background jobs by typing **`jobs`**. However, its stdin, stdout, stderr are still joined to the terminal.
<!--more-->

```
sudo rsync Templates/* /var/www/html/files/ &
$ jobs
$ disown  -h  %1
# or
$ disown  -a

$ jobs
```

```
$ nohup tar -czf iso.tar.gz Templates/* &
$ jobs
```