---
layout: post
title: Install TigerVNC Server/Client on Ubuntu Machines
date: 2024-06-30
description: Install TigerVNC Server/Client on Ubuntu Machines
tags: linux, ubuntu, commands
comments: true
---
# Introduction
TigerVNC is a client/server application that allows users to launch and interact with graphical applications on remote machines. TigerVNC provides the levels of performance necessary to run 3D and video applications, and it attempts to maintain a common look and feel and re-use components, where possible, across the various platforms that it supports.


<br>
# Install and Setup TigerVNC Viewer on Server

<br>
#### Step 1. Install TigerVNC Server.

First of all, ssh to the server with administrator account with sudo privileges, and install the packages using the following commands on the server.

```bash
$ sudo apt update
$ sudo apt install xfce4 xfce4-goodies
$ sudo apt install tightvncserver

```

<br>
#### Step 2. Setup TigerVNC Server

SSH to the server

```bash
$ ssh xxx@yyy.zzz
```

Configure VNC Server
```bash
$ mkdir ~/.vnc
$ vim ~/.vnc/xstartup
```

Copy the following command to your ~/.vnc/xstartup
```bash
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```


The, add execution permission to ~/.vnc/xstartup
```bash
$ chmod +x ~/.vnc/xstartup
```

<br>
#### Step 3. Launch VNC Server

To launch VNC server, run the command below:

> [Note]  
> The password must be between six and eight characters long. Passwords more than 8 characters will be truncated automatically.

```bash
# configure the resolution by -geometry
$ vncserver :1 -geometry 2560x1440
Output
You will require a password to access your desktops.

Password:
Verify:
Would you like to enter a view-only password (y/n)? n
xauth:  file /home/hc29225/.Xauthority does not exist

New 'X' desktop is borg:1

Creating default startup script /home/hc29225/.vnc/xstartup
Starting applications specified in /home/hc29225/.vnc/xstartup
Log file is /home/hc29225/.vnc/borg:1.log
```

<br>
#### Useful commands
Note that if you ever want to change your password or add a view-only password, you can do so with the vncpasswd command:
```bash
$ vncpasswd
```

To list current vncserver processes
```bash
$ vncserver -list
```


To kill the vncserver process on the remote
```bash
$ vncserver -kill :1
```



<br>
# Install and Setup TigerVNC Viewer on Local

<br>
#### Step 1. Install tigervnc-viewer
Please run the script to install tigervnc-viewer on your local machine. Note that you should have the sudo privilege of your machine or the machine is pre-installed with tigervnc-viewer.

```bash
$ sudo apt install tigervnc-viewer
```


<br>
#### Step 2. Setup TigerVNC Viewer on Local
After installing the TigerVNC on the client, you should be able to find it in our machine. You will see the configuration window after clicking the TigerVNC Viewer.

<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/tigervnc/tigervnc_local.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>



<br>
#### Step 3. Connect to VNC Server

Ssh to the server with port forwarding
```bash
$ ssh -L 59000:localhost:5901 -C -N -l account xxx.yyy.zzz
```


Or, you can configure your ~/.ssh/config like below then doing ssh
```bash
# in your ~/.ssh/config
Host xxx
    HostName xxx.yyy.zzz
    User account
    IdentityFile ~/.ssh/id_rsa
    LocalForward 59000 localhost:5901
# then, ssh to the server
$ ssh account@xxx
```



After ssh to the server in a terminal, Launch VNC Viewer and type in localhost:59000 and vnc password
<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/tigervnc/tigervnc_login.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>
You should be able to see the graphical user interface once connected
<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/tigervnc/tigervnc_desktop.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>


<br>
# Troubleshooting

The gray screen is forwarded by TigerVNC 
<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/tigervnc/tigervnc_desktop_grey.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

Solution: Add executable permission ~/.vnc/xstartup on the server to fix this issue
```bash
$ chmod +x ~/.vnc/xstartup
```


<br>
# Reference:
- [https://tigervnc.org/](https://tigervnc.org/)
- [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-22-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-22-04)
- [https://www.howtoforge.com/how-to-install-vnc-server-ubuntu-22-04/](https://www.howtoforge.com/how-to-install-vnc-server-ubuntu-22-04/)
- [https://bytexd.com/how-to-install-configure-vnc-server-on-ubuntu/](https://bytexd.com/how-to-install-configure-vnc-server-on-ubuntu/)