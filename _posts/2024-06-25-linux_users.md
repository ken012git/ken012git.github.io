---
layout: post
title: Create/Delete Users on Linux Machines
date: 2024-06-25
description: Create/Delete Users on Linux Machines
tags: linux, ubuntu, commands
comments: true
---
# Create Users 
Create user with default using bash shell `-s` (`--shell`) and home directory `-m` (`--create-home`)
```bash
$ sudo useradd -m -s /bin/bash <usrname>
$ sudo passwd <usrname>
```

# Delete Users
Delete that user's home directory and mail spool by using the `-r` flag with the command
```bash
$ sudo userdel -r <usrname>
```

# Reference:
- [https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command/](https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command/)
- [https://sg.godaddy.com/help/remove-a-linux-user-19158](https://sg.godaddy.com/help/remove-a-linux-user-19158)

