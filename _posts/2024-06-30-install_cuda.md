---
layout: post
title: Install/Update CUDA on Ubuntu Machines
date: 2024-06-30
description: Install/Update CUDA on Ubuntu Machines
tags: linux, ubuntu, nvidia, commands
comments: true
---
# Introduction
This document contain the step to install/update CUDA on Ubuntu machines


# Installation 

#### Step 1. Download cuda from NVIDIA official [website](https://developer.nvidia.com/cuda-downloads). Either Deb (local), deb (network), or runfile (local) would work.

<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/install_cuda/cuda.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

#### Step 2. Follow the instructions on the NVIDIA official website to download CUDA.

<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/install_cuda/install_cuda.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

#### Step 3. Reboot the machine
```bash
$ sudo reboot
```
<br>

#### Step 4. Check system cuda after reboot
<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/install_cuda/check_cuda.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

#### Step 5. Setup the environment in ~/.bashrc. Please see the official [document](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#environment-setup).

```bash
### add cuda path in ~/.bashrc
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
<br>

#### Step 6. Source new environment
```bash
$ source ~/.bashrc
```
<br>

#### Step 7. Test driver
```bash
$ nvidia-smi
```
<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/install_cuda/test_driver.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>

#### Step 8. Test cuda compiler
```bash
$ nvcc -V
```
<br>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/install_cuda/test_cudnn.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br>


# Reference:
- [https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
- [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)
- [https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive)
- [https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#environment-setup](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#environment-setup)
