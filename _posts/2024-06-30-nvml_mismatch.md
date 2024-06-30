---
layout: post
title: "NVML: Driver/library Version Mismatch"
date: 2024-06-30
description: "Solving NVML: Driver/library Version Mismatch"
tags: linux, ubuntu, nvidia, commands
comments: true
---
# Introduction
The Ubuntu kernel and Nvidia driver will be updated automatically and sometimes cause driver mismatch issues on the system. This document shows how to solve the Nvidia driver mismatch problem on a Ubuntu OS.

> [TL;DR]  
> The first thing you need to try is rebooting the machine!

If rebooting does not work for your case, please try the following steps.

# Issue
```bash
$ nvidia-smi
Failed to initialize NVML: Driver/library version mismatch
NVML library version: 550.54
```
Check kernel message
```bash
sudo dmesg
...
[ 1865.855970] NVRM: API mismatch: the client has the version 550.54.14, but
               NVRM: this kernel module has the version 545.23.08.  Please
               NVRM: make sure that this kernel module and all NVIDIA driver
               NVRM: components have the same version.
...
```

# Solution 

<br>
#### Step 1. Check system driver and process driver
- The current running driver is `545.23.08`
```bash
$ cat /proc/driver/nvidia/version
NVRM version: NVIDIA UNIX x86_64 Kernel Module  545.23.08  Mon Nov  6 23:49:37 UTC 2023
GCC version:  gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04) 
```

- The system driver in the Dynamic Kernel Module is `545.23.08`
```bash
$ dkms status
nvidia/545.23.08, 5.15.0-94-generic, x86_64: installed
nvidia/545.23.08, 5.15.0-97-generic, x86_64: installed
$ modinfo nvidia | grep ^version
version:        545.23.08
```

<br>
#### Step 2. Check available drivers

`nvidia-driver-550 - third-party non-free recommended` is the recommended driver.

```bash
$ sudo ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:01.1/0000:01:00.0 ==
modalias : pci:v000010DEd00002231sv000010DEsd0000147Ebc03sc00i00
vendor   : NVIDIA Corporation
model    : GA102GL [RTX A5000]
manual_install: True
driver   : nvidia-driver-535-server-open - third-party non-free
driver   : nvidia-driver-525-server - third-party non-free
driver   : nvidia-driver-535-server - third-party non-free
driver   : nvidia-driver-545-open - third-party non-free
driver   : nvidia-driver-470-server - third-party non-free
driver   : nvidia-driver-535-open - third-party non-free
driver   : nvidia-driver-525-open - third-party non-free
driver   : nvidia-driver-550-open - third-party non-free
driver   : nvidia-driver-550 - third-party non-free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin
```

<br>
#### Step 3. Try to install the recommended driver first

```bash
$ sudo apt install nvidia-driver-550 nvidia-dkms-550
```

<br>
#### Step 4. Solving the error: trying to overwrite
If you get this error

```bash
Unpacking libnvidia-gl-550:amd64 (550.54.14-0ubuntu1) ...
dpkg: error processing archive /var/cache/apt/archives/libnvidia-gl-550_550.54.14-0ubuntu1_amd64.deb (--unpack):
 trying to overwrite '/usr/lib/x86_64-linux-gnu/libnvidia-api.so.1', which is also in package libnvidia-extra-545:amd64 545.23.08-0ubuntu1
dpkg-deb: error: paste subprocess was killed by signal (Broken pipe)
Errors were encountered while processing:
```

Try to force install the driver. There will be an error at the end of the message, but this should be fine.

```bash
$ sudo dpkg -i --force-overwrite /var/cache/apt/archives/libnvidia-gl-550_550.54.14-0ubuntu1_amd64.deb 
(Reading database ... 292518 files and directories currently installed.)
Preparing to unpack .../libnvidia-gl-550_550.54.14-0ubuntu1_amd64.deb ...
dpkg-query: no packages found matching libnvidia-gl-535
Unpacking libnvidia-gl-550:amd64 (550.54.14-0ubuntu1) ...
dpkg: warning: overriding problem because --force enabled:
dpkg: warning: trying to overwrite '/usr/lib/x86_64-linux-gnu/libnvidia-api.so.1', which is also in package libnvidia-extra-545:amd64 545.23.08-0ubuntu1
dpkg: dependency problems prevent configuration of libnvidia-gl-550:amd64:
 libnvidia-gl-550:amd64 depends on libnvidia-common-550; however:
  Package libnvidia-common-550 is not configured yet.
 libnvidia-gl-550:amd64 depends on libnvidia-compute-550 (>= 550.54.14); however:
  Package libnvidia-compute-550:amd64 is not configured yet.

dpkg: error processing package libnvidia-gl-550:amd64 (--install):
 dependency problems - leaving unconfigured
Processing triggers for libc-bin (2.35-0ubuntu3.6) ...
Errors were encountered while processing:
 libnvidia-gl-550:amd64
```

Then, fix the broken dependency.

```bash
(base) lab-admin@hopper:~$ sudo apt --fix-broken install
....
depmod...
Setting up libnvidia-encode-550:amd64 (550.54.14-0ubuntu1) ...
Setting up nvidia-driver-550 (550.54.14-0ubuntu1) ...
Setting up cuda-drivers-550 (550.54.14-1) ...
Setting up cuda-drivers (550.54.14-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.6) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for initramfs-tools (0.140ubuntu13.4) ...
update-initramfs: Generating /boot/initrd.img-5.15.0-97-generic
W: Possible missing firmware /lib/firmware/ast_dp501_fw.bin for module ast
Package profile updates
        status: 1
        updates: []
        exceptions: 
        
needrestart is being skipped since dpkg has failed
```

Check the recommended driver is fixed

```bash
(base) lab-admin@hopper:~$ sudo apt --fix-broken install nvidia-driver-550 nvidia-dkms-550
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
nvidia-dkms-550 is already the newest version (550.54.14-0ubuntu1).
nvidia-dkms-550 set to manually installed.
nvidia-driver-550 is already the newest version (550.54.14-0ubuntu1).
nvidia-driver-550 set to manually installed.
The following packages were automatically installed and are no longer required:
  libwmf0.2-7 nsight-compute-2023.3.0
Use 'sudo apt autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
```

But now if you try the driver, it will show the same error. (Not sure if we restart the machine now will fix it) 
```bash
$ nvidia-smi
Failed to initialize NVML: Driver/library version mismatch
NVML library version: 550.54
```

<br>
#### Step 5. Purge all nvidia drivers and re-install
We will continue to purge all nvidia drivers first and re-install it again to keep our system environment simple. As you can see, there are lots of drivers in the system, for example nvidia-dkms-xxx

```bash
$ dpkg -l | grep nvidia
ii  libnvidia-cfg1-550:amd64                          550.54.14-0ubuntu1                      amd64        NVIDIA binary OpenGL/GLX configuration library
ii  libnvidia-common-550                              550.54.14-0ubuntu1                      all          Shared files used by the NVIDIA libraries
rc  libnvidia-compute-515:amd64                       515.105.01-0ubuntu0.22.04.1             amd64        NVIDIA libcompute package
rc  libnvidia-compute-515-server:amd64                515.65.01-0ubuntu0.22.04.1              amd64        NVIDIA libcompute package
rc  libnvidia-compute-530:amd64                       530.30.02-0ubuntu1                      amd64        NVIDIA libcompute package
rc  libnvidia-compute-545:amd64                       545.23.08-0ubuntu1                      amd64        NVIDIA libcompute package
ii  libnvidia-compute-550:amd64                       550.54.14-0ubuntu1                      amd64        NVIDIA libcompute package
ii  libnvidia-container-tools                         1.14.5-1                                amd64        NVIDIA container runtime library (command-line tools)
ii  libnvidia-container1:amd64                        1.14.5-1                                amd64        NVIDIA container runtime library
ii  libnvidia-decode-550:amd64                        550.54.14-0ubuntu1                      amd64        NVIDIA Video Decoding runtime libraries
ii  libnvidia-encode-550:amd64                        550.54.14-0ubuntu1                      amd64        NVENC Video Encoding runtime library
ii  libnvidia-extra-550:amd64                         550.54.14-0ubuntu1                      amd64        Extra libraries for the NVIDIA driver
ii  libnvidia-fbc1-550:amd64                          550.54.14-0ubuntu1                      amd64        NVIDIA OpenGL-based Framebuffer Capture runtime library
ii  libnvidia-gl-550:amd64                            550.54.14-0ubuntu1                      amd64        NVIDIA OpenGL/GLX/EGL/GLES GLVND libraries and Vulkan ICD
rc  linux-modules-nvidia-515-server-5.15.0-47-generic 5.15.0-47.51                            amd64        Linux kernel nvidia modules for version 5.15.0-47
rc  linux-objects-nvidia-515-server-5.15.0-47-generic 5.15.0-47.51                            amd64        Linux kernel nvidia modules for version 5.15.0-47 (objects)
rc  nvidia-compute-utils-515                          515.105.01-0ubuntu0.22.04.1             amd64        NVIDIA compute utilities
rc  nvidia-compute-utils-515-server                   515.65.01-0ubuntu0.22.04.1              amd64        NVIDIA compute utilities
rc  nvidia-compute-utils-530                          530.30.02-0ubuntu1                      amd64        NVIDIA compute utilities
rc  nvidia-compute-utils-545                          545.23.08-0ubuntu1                      amd64        NVIDIA compute utilities
ii  nvidia-compute-utils-550                          550.54.14-0ubuntu1                      amd64        NVIDIA compute utilities
ii  nvidia-container-toolkit                          1.14.5-1                                amd64        NVIDIA Container toolkit
ii  nvidia-container-toolkit-base                     1.14.5-1                                amd64        NVIDIA Container Toolkit Base
rc  nvidia-cuda-toolkit                               11.5.1-1ubuntu1                         amd64        NVIDIA CUDA development toolkit
rc  nvidia-dkms-515                                   515.105.01-0ubuntu0.22.04.1             amd64        NVIDIA DKMS package
rc  nvidia-dkms-530                                   530.30.02-0ubuntu1                      amd64        NVIDIA DKMS package
rc  nvidia-dkms-545                                   545.23.08-0ubuntu1                      amd64        NVIDIA DKMS package
ii  nvidia-dkms-550                                   550.54.14-0ubuntu1                      amd64        NVIDIA DKMS package
ii  nvidia-docker2                                    2.13.0-1                                all          nvidia-docker CLI wrapper
ii  nvidia-driver-550                                 550.54.14-0ubuntu1                      amd64        NVIDIA driver metapackage
ii  nvidia-firmware-550-550.54.14                     550.54.14-0ubuntu1                      amd64        Firmware files used by the kernel module
rc  nvidia-kernel-common-515                          515.105.01-0ubuntu0.22.04.1             amd64        Shared files used with the kernel module
rc  nvidia-kernel-common-515-server                   515.65.01-0ubuntu0.22.04.1              amd64        Shared files used with the kernel module
rc  nvidia-kernel-common-530                          530.30.02-0ubuntu1                      amd64        Shared files used with the kernel module
rc  nvidia-kernel-common-545                          545.23.08-0ubuntu1                      amd64        Shared files used with the kernel module
ii  nvidia-kernel-common-550                          550.54.14-0ubuntu1                      amd64        Shared files used with the kernel module
ii  nvidia-kernel-source-550                          550.54.14-0ubuntu1                      amd64        NVIDIA kernel source package
ii  nvidia-modprobe                                   545.23.08-0ubuntu1                      amd64        Load the NVIDIA kernel driver and create device files
ii  nvidia-prime                                      0.8.17.1                                all          Tools to enable NVIDIA's Prime
ii  nvidia-settings                                   545.23.08-0ubuntu1                      amd64        Tool for configuring the NVIDIA graphics driver
ii  nvidia-utils-550                                  550.54.14-0ubuntu1                      amd64        NVIDIA driver support binaries
ii  screen-resolution-extra                           0.18.2                                  all          Extension for the nvidia-settings control panel
ii  xserver-xorg-video-nvidia-550                     550.54.14-0ubuntu1                      amd64        NVIDIA binary Xorg driver
```

Remove all nvidia related packages.

```bash
$ sudo apt --purge remove *nvidia*

The following packages were automatically installed and are no longer required:
  dctrl-tools dkms libgdk-pixbuf-xlib-2.0-0 libgdk-pixbuf2.0-0 libwmf0.2-7 nsight-compute-2023.3.0 pkg-config screen-resolution-extra
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  cuda* cuda-12-3* cuda-demo-suite-12-3* cuda-drivers* cuda-drivers-550* cuda-runtime-12-3* libnvidia-cfg1-550* libnvidia-common-550*
  libnvidia-compute-515* libnvidia-compute-515-server* libnvidia-compute-530* libnvidia-compute-545* libnvidia-compute-550*
  libnvidia-container-tools* libnvidia-container1* libnvidia-decode-550* libnvidia-encode-550* libnvidia-extra-550* libnvidia-fbc1-550*
  libnvidia-gl-550* linux-modules-nvidia-515-server-5.15.0-47-generic* linux-objects-nvidia-515-server-5.15.0-47-generic*
  nvidia-compute-utils-515* nvidia-compute-utils-515-server* nvidia-compute-utils-530* nvidia-compute-utils-545* nvidia-compute-utils-550*
  nvidia-container-toolkit* nvidia-container-toolkit-base* nvidia-cuda-toolkit* nvidia-dkms-515* nvidia-dkms-530* nvidia-dkms-545*
  nvidia-dkms-550* nvidia-docker2* nvidia-driver-550* nvidia-firmware-550-550.54.14* nvidia-kernel-common-515* nvidia-kernel-common-515-server*
  nvidia-kernel-common-530* nvidia-kernel-common-545* nvidia-kernel-common-550* nvidia-kernel-source-550* nvidia-modprobe* nvidia-prime*
  nvidia-settings* nvidia-utils-550* xserver-xorg-video-nvidia-550*
0 upgraded, 0 newly installed, 48 to remove and 2 not upgraded.
After this operation, 751 MB disk space will be freed.
Do you want to continue? [Y/n]  Y
```


Check the environment again to make sure there is no NVIDIA related packages
```bash
$ dpkg -l | grep nvidia
ii  screen-resolution-extra                    0.18.2                                  all          Extension for the nvidia-settings control panel
```

<br>
#### Step 6. Re-install the recommended driver

```bash
$ sudo apt install nvidia-driver-550 nvidia-dkms-550
```

Check the environment again after the recommended driver is installed.

```bash
$ dpkg -l | grep nvidia
ii  libnvidia-cfg1-550:amd64                   550.54.14-0ubuntu1                      amd64        NVIDIA binary OpenGL/GLX configuration library
ii  libnvidia-common-550                       550.54.14-0ubuntu1                      all          Shared files used by the NVIDIA libraries
ii  libnvidia-compute-550:amd64                550.54.14-0ubuntu1                      amd64        NVIDIA libcompute package
ii  libnvidia-decode-550:amd64                 550.54.14-0ubuntu1                      amd64        NVIDIA Video Decoding runtime libraries
ii  libnvidia-encode-550:amd64                 550.54.14-0ubuntu1                      amd64        NVENC Video Encoding runtime library
ii  libnvidia-extra-550:amd64                  550.54.14-0ubuntu1                      amd64        Extra libraries for the NVIDIA driver
ii  libnvidia-fbc1-550:amd64                   550.54.14-0ubuntu1                      amd64        NVIDIA OpenGL-based Framebuffer Capture runtime library
ii  libnvidia-gl-550:amd64                     550.54.14-0ubuntu1                      amd64        NVIDIA OpenGL/GLX/EGL/GLES GLVND libraries and Vulkan ICD
ii  nvidia-compute-utils-550                   550.54.14-0ubuntu1                      amd64        NVIDIA compute utilities
ii  nvidia-dkms-550                            550.54.14-0ubuntu1                      amd64        NVIDIA DKMS package
ii  nvidia-driver-550                          550.54.14-0ubuntu1                      amd64        NVIDIA driver metapackage
ii  nvidia-firmware-550-550.54.14              550.54.14-0ubuntu1                      amd64        Firmware files used by the kernel module
ii  nvidia-kernel-common-550                   550.54.14-0ubuntu1                      amd64        Shared files used with the kernel module
ii  nvidia-kernel-source-550                   550.54.14-0ubuntu1                      amd64        NVIDIA kernel source package
ii  nvidia-prime                               0.8.17.1                                all          Tools to enable NVIDIA's Prime
ii  nvidia-settings                            550.54.14-0ubuntu1                      amd64        Tool for configuring the NVIDIA graphics driver
ii  nvidia-utils-550                           550.54.14-0ubuntu1                      amd64        NVIDIA driver support binaries
ii  screen-resolution-extra                    0.18.2                                  all          Extension for the nvidia-settings control panel
ii  xserver-xorg-video-nvidia-550              550.54.14-0ubuntu1                      amd64        NVIDIA binary Xorg driver
```

But now if you try the driver, it will show the same error.  We need to reboot the machine first to reload the driver.

```bash
$ nvidia-smi
Failed to initialize NVML: Driver/library version mismatch
NVML library version: 550.54
```

Reboot

```bash
$ sudo reboot
```

<br>
#### Step 7 (Optional) Keep the hernel version

> [Warning]  
> This will prevent the machine from some important updates

If we want to keep the version and prevent this happening again, we could use 
```bash
`apt-mark hold` 
$ sudo apt-mark hold nvidia-dkms-550: 
nvidia-dkms-550 set on hold.
$ sudo apt-mark hold nvidia-driver-550
nvidia-driver-550 set on hold.
$ sudo apt-mark hold nvidia-utils-550
nvidia-utils-550 set on hold.
$ uname -r
5.15.0-97-generic
$ sudo apt-mark hold 5.15.0-97-generic # hold the kernel
```

# Reference:
- [https://stackoverflow.com/questions/74151409/nvidia-nvml-driver-library-version-mismatch-dkms-modules-drivers-and-modinfo](https://stackoverflow.com/questions/74151409/nvidia-nvml-driver-library-version-mismatch-dkms-modules-drivers-and-modinfo)
- [https://stackoverflow.com/questions/43022843/nvidia-nvml-driver-library-version-mismatch/67105064#67105064](https://stackoverflow.com/questions/43022843/nvidia-nvml-driver-library-version-mismatch/67105064#67105064)
- [https://forums.developer.nvidia.com/t/problem-with-apt-and-nvidia-440-on-ubuntu-20-04/115281/2](https://forums.developer.nvidia.com/t/problem-with-apt-and-nvidia-440-on-ubuntu-20-04/115281/2)
- [https://askubuntu.com/questions/1436506/how-to-resolve-unmet-dependencies-error-when-upgrading-depends-nvidia-kernel-c](https://askubuntu.com/questions/1436506/how-to-resolve-unmet-dependencies-error-when-upgrading-depends-nvidia-kernel-c)
- [https://askubuntu.com/questions/938494/how-to-i-prevent-ubuntu-from-kernel-version-upgrade-and-notification](https://askubuntu.com/questions/938494/how-to-i-prevent-ubuntu-from-kernel-version-upgrade-and-notification)
