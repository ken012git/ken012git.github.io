---
layout: post
title: Install Rootless Docker on Ubuntu Machines
date: 2024-06-25
description: Install Rootless Docker on Ubuntu Machines
tags: linux, ubuntu, docker, nvidia, commands
comments: true
---

# Introduction
Running docker usually requires root privileges. This document contains the rootless docker installation steps on Ubuntu machines. Note that the root privilege is required during the installation.

# Installation


## Step 1 Install uidmap (sudo required)
Step 6 Install uidmap for Rootless Docker
```bash
$ sudo apt-get install -y uidmap
```

## Step 2 Setup User group (sudo required)

Follow the [document](https://rootlesscontaine.rs/getting-started/common/subuid/) to setup the user group. If `subuids` and `subgids` are not configured, you need to edit /etc/subuid and /etc/subgid directly with a text editor

```bash
$ cat /etc/subuid
user1:100000:65536
$ cat /etc/subgids
user1:100000:65536
```

## Step 3 Configure NVIDIA Docker for User Group (sudo required)
> [!WARNING]  
> Warning: the side effect of the setting is that using nvidia docker with sudo will cause error: Failed to initialize NVML: Unknown Error.

```bash
$ sudo vim /etc/nvidia-container-runtime/config.toml
# set no-cgroups = true
no-cgroups = true
```


## Step 4. Setup Rootless Docker
```bash
$ dockerd-rootless-setuptool.sh install
```

## Step 5. Restart Docker
```bash
$ systemctl --user start docker
$ systemctl --user enable docker
# export DOCKER_HOST or add to ~/.bashrc and source ~/.bashrc
$ export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
```

## Step 6. Test Docker without Sudo
```bash
# without sudo to run docker
$ docker run hello-world
# without sudo to run docker and access gpu 
$ docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```









# Reference:
- [https://github.com/NVIDIA/nvidia-docker/issues/1447](https://github.com/NVIDIA/nvidia-docker/issues/1447)
- [https://github.com/NVIDIA/nvidia-docker/issues/1672](https://github.com/NVIDIA/nvidia-docker/issues/1672)
- [https://github.com/NVIDIA/nvidia-docker/issues/1155](https://github.com/NVIDIA/nvidia-docker/issues/1155)


