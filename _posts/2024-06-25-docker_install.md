---
layout: post
title: Install Nvidia Docker on Ubuntu Machines
date: 2024-06-25
description: Install Nvidia Docker on Ubuntu Machines
tags: linux, ubuntu, docker, nvidia, commands
comments: true
---

# Introduction
To run Docker with accessibility of GPU and CUDA support, NVIDIA Docker has to be installed on the machine. Docker is a tool that encapsulates the environments, e.g, CUDA, CUDNN and all kinds of libraries and dependencies in a container without messing up the machine's environment. This is especially beneficial when we are duplicating some methods that use other packages e.g., Tensorflow Pytorch, Keras, Caffe, MXNet., or depending on old/new CUDA and CUDNN versions. 

# Installation

> [!WARNING]  
> Warning: we will remove the Docker originally installed on the Machines by SNAP, and re-install capable Docker using apt-get

## Step 1 remove docker install from SNAP
Remove docker
```bash
$ sudo snap remove --purge docker
2022-10-03T18:37:22Z INFO Waiting for "snap.docker.dockerd.service" to stop.
docker removed
```

To check if there is still docker installed by apt-get
```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package 'docker-engine' is not installed, so not removed
Package 'containerd' is not installed, so not removed
Package 'runc' is not installed, so not removed
Package 'docker' is not installed, so not removed
Package 'docker.io' is not installed, so not removed
0 upgraded, 0 newly installed, 0 to remove and 42 not upgraded.
```

## Step 2 Install Docker from APT
Follow the steps in Docker official document: https://docs.docker.com/engine/install/ubuntu/

## Step 3 Reboot server

```bash
$ sudo reboot
```

## Step 4 Test Docker with sudo

```bash
$ sudo service docker start
$ sudo docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:62af9efd515a25f84961b70f973a798d2eca956b1b2b026d0a4a63a3b0b6a3f2
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```


## Step 5 Install NVIDIA Docker
Follow the steps in NVIDIA Docker official document: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker

## Step 6 Test NVIDIA Docker with sudo
```bash
$ sudo  docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```


# Reference:
- [https://github.com/NVIDIA/nvidia-docker/issues/1447](https://github.com/NVIDIA/nvidia-docker/issues/1447)
- [https://github.com/NVIDIA/nvidia-docker/issues/1672](https://github.com/NVIDIA/nvidia-docker/issues/1672)
- [https://github.com/NVIDIA/nvidia-docker/issues/1155](https://github.com/NVIDIA/nvidia-docker/issues/1155)


