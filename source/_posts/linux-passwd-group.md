---
title: Linux passwd and group Mangagement
date: 2020-01-23 10:14:46
categories:
tags:
---
# Format

## /etc/passwd
> Username, Password, User ID, Group ID, User ID Info, Home directory, Command/shell

Example:

> tom:x:1001:1001:Tom Cruise:/home/zac:/bin/bash

## /etc/group
> Group name, Password, Group ID, Group List

Example:

> sudo:x:27:Lucy,Tom

<!--more-->
# User and Group Management Files

> **`/etc/passwd`** and **`/etc/group`**


# Add

``` sh
sudo adduser new_username
# Or
sudo useradd new_username
```

# Delete
``` sh
sudo userdel username
# Then you may want to delete the home directory for the deleted user account :
sudo rm -r /home/username
```

# Edit
To modify the username of a user:

`usermod -l new_username old_username`

To change the password for a user:

`sudo passwd username`

To change the shell for a user:

`sudo chsh username`

To change the details for a user (for example real name):

`sudo chfn username`

To add a user to the sudo group:

* `adduser username sudo`

* `usermod -aG sudo username`