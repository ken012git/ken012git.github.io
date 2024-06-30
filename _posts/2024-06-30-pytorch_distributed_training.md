---
layout: post
title: Distributed Training with Pytorch
date: 2024-06-30
description: Distributed Training with Pytorch
tags: linux, ubuntu, nvidia, commands, coding, pytorch
comments: true
---

# Introduction
Distributed training is necessary for large models training tasks such as neural architecture search supernet, diffusion model or large language models. This document contains the steps to configure machines and launch distributed training tasks with Pytorch framework on GPU servers.

<br>
# Server Configuration 

#### Step 1 Disable firewall and IPV6
> [WARNING]
> This step may expose the machines under certain threats and attacks.

Disable firewall

```bash
$ sudo ufw status
Status: active
$ sudo ufw disable
$ sudo ufw status
Status: inactive
```

Disable IPV6:
```bash
$ sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
$ sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```


#### Step 2. Export NCCL SOCKET

Some of our machines do not use the default **eth0**, so we have to configure the `NCCL_SOCKET_IFNAME` correctly on **all** the machines. 

To check the socket name.

```bash
$ ifconfig     
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500                            
        inet xx.xx.xx.xx  netmask 255.255.0.0  broadcast xx.xx.xx.xx          
        ether 02:42:fe:49:77:fe  txqueuelen 0  (Ethernet)                        
        RX packets 0  bytes 0 (0.0 B)                                            
        RX errors 0  dropped 0  overruns 0  frame 0                              
        TX packets 0  bytes 0 (0.0 B)                                            
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0               
                                                                                 
eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500                       
        inet xx.xx.123.214  netmask 255.255.255.192  broadcast xx.xx.xx.xx   
        ether 3c:ec:ef:05:1f:ca  txqueuelen 1000  (Ethernet)                     
        RX packets 478564990  bytes 700182821494 (700.1 GB)                      
        RX errors 0  dropped 6  overruns 2135  frame 0                           
        TX packets 880702  bytes 274251519 (274.2 MB)                            
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0               
        device memory 0xc1320000-c133ffff                                        
                                                                                 
eno2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500                       
        inet xx.xx.123.215  netmask 255.255.255.192  broadcast xx.xx.xx.xx  
        ether 3c:ec:ef:05:1f:cb  txqueuelen 1000  (Ethernet)                     
        RX packets 136141633  bytes 187956572012 (187.9 GB)                      
        RX errors 0  dropped 6  overruns 2335  frame 0                           
        TX packets 178699579  bytes 168433035752 (168.4 GB)                      
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0               
        device memory 0xc1300000-c131ffff         
```

Install `net-tools` if you see the error
```bash
Command 'ifconfig' not found, but can be installed with:
apt install net-tools
Please ask your administrator.
```


Export the corresponding socket name
```bash
# Both eno1 and eno2 should work.
$ export NCCL_SOCKET_IFNAME=eno2
```

<br>
#### Step 3. Launch Distributed Training

Get the IP addresses of the machines in the cluster. Note that the ip address should match the NCCL_SOCKET_IFNAME that we export. For example:

```bash
lovelace: xx.xx.120.112
hopper: xx.xx.121.104
borg: xx.xx.122.109
johnson: xx.xx.123.214
allen: xx.xx.124.214
```


<br>
#### Step 4. Run training scripts

Lei Mao wrote a good [toy example](https://leimao.github.io/blog/PyTorch-Distributed-Training/) where we could start with. Here we use 2 machines (--nnodes) as an example. In each node, 4 GPUs (--nproc-per-node) will be used. Therefore, the world size is 8 (8 GPUs in total). 

```bash
# Host: (--node-rank 0)
torchrun --nnodes=2 --nproc-per-node=4 --node-rank=0 \
        --rdzv-id=456 --rdzv-backend=c10d   \
        --rdzv-endpoint=xx.xx.120.112:29500 \
        train.py --param1 xxx --param2 yyy …
```


```bash
# Client: (--node-rank 1)
torchrun --nnodes=2 --nproc-per-node=4 --node-rank=1 \
        --rdzv-id=456 --rdzv-backend=c10d   \
        --rdzv-endpoint=xx.xx.120.112:29500 \        
        train.py --param1 xxx --param2 yyy …

```



<br>
# Network Bandwidth
Pytorch relies on the Internet connection to pass the updated weights. Therefore, a high bandwidth machine that serves as the master will speed up the training.


Test Bandwidth without sudo
```bash
$ curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
...
...
Testing download speed................................................................................
Download: 88.85 Mbit/s
Testing upload speed......................................................................................................
Upload: 11.87 Mbit/s
```

Test Bandwidth with sudo privilege 
With sudo privilege, we can test a specific network interface, for example enp194s0f0
```bash
$ sudo ethtool enp194s0f0 | grep -i speed
        Speed: 10000Mb/s
```


<br>
#### Bandwidth Monitoring Tools

Bmon is a good internet bandwidth monitoring tool. Use the command: `sudo apt install bmon` to install the tool. More tools are available here.

Here is a snapshot of the distributed training. The weights are passing through the enp194s0f1.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/pytorch_distributed_training/bandwidth.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>


<br>
# Troubleshooting

<br>
#### Hostname not known [Issue](https://github.com/pytorch/pytorch/issues/74824#issuecomment-1500144250)

Error message:

```bash
[W socket.cpp:601] [c10d] The IPv6 network addresses of (ece-a58489.austin.utexas.edu, 44685) cannot be retrieved (gai error: -2 - Name or service not known).
```

Solution: add hostname and ip to /etc/hosts. It requires sudo privileges. 

```bash
$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 ece-a55028
xx.xx.120.112 ece-a58489.yyy.zzz # add ip for lovelace

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

<br>
#### Debugging

To dump more information for debugging the distributed training, we have to export a few environment variables.

```bash
$ export NCCL_DEBUG=INFO
$ export TORCH_CPP_LOG_LEVEL=INFO
$ export TORCH_DISTRIBUTED_DEBUG=INFO
```



Debugging message:
 
```bash
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
master_addr is only used for static rdzv_backend and when rdzv_endpoint is not specified.                                                                                     
WARNING:torch.distributed.run:                                                                                                                                                
*****************************************                                                                                                                                     
Setting OMP_NUM_THREADS environment variable for each process to be 1 in default, to avoid your system being overloaded, please further tune the variable for optimal performa
nce in your application as needed.                                                                                                                                            
*****************************************                                                                                                                                     
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (10.157.244.213, 29500).                                                    
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:29500 on [johnson.ece.utexas.edu]:46672.                                                  
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (10.157.244.213, 29500).                                                    
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:29500 on [johnson.ece.utexas.edu]:46682.                                                  
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
[I debug.cpp:49] [c10d] The debug level is set to INFO.                                                                                                                       
2023-10-09 23:39:49.119410: I tensorflow/core/platform/cpu_feature_guard.cc:182] This TensorFlow binary is optimized to use available CPU instructions in performance-critical
 operations.                                                                                                                                                                  
To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.                                                  
2023-10-09 23:39:49.170222: I tensorflow/core/platform/cpu_feature_guard.cc:182] This TensorFlow binary is optimized to use available CPU instructions in performance-critical
 operations.                                                                                                                                                                  
To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.                                                  
2023-10-09 23:39:49.170405: I tensorflow/core/platform/cpu_feature_guard.cc:182] This TensorFlow binary is optimized to use available CPU instructions in performance-critical
 operations.                                                                                                                                                                  
To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.                                                  
2023-10-09 23:39:49.170545: I tensorflow/core/platform/cpu_feature_guard.cc:182] This TensorFlow binary is optimized to use available CPU instructions in performance-critical
 operations.                                                                                                                                                                  
To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.                                                  
2023-10-09 23:39:49.851032: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] TF-TRT Warning: Could not find TensorRT                                         [488/1839]
2023-10-09 23:39:49.855787: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] TF-TRT Warning: Could not find TensorRT
2023-10-09 23:39:49.965667: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] TF-TRT Warning: Could not find TensorRT
| distributed init (world_size 24, global rank 20, local gpu id 4): env://
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (allen.local, 32927).
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:32927 on [johnson.ece.utexas.edu]:35816.
| distributed init (world_size 24, global rank 18, local gpu id 2): env://
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (allen.local, 32927).
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:32927 on [johnson.ece.utexas.edu]:35822.
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (allen.local, 32927).
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:32927 on [johnson.ece.utexas.edu]:35834.
[I ProcessGroupNCCL.cpp:665] [Rank 20] ProcessGroupNCCL initialized with following options:
NCCL_ASYNC_ERROR_HANDLING: 1
NCCL_DESYNC_DEBUG: 0
NCCL_BLOCKING_WAIT: 0
TIMEOUT(ms): 1800000
USE_HIGH_PRIORITY_STREAM: 0
[I ProcessGroupNCCL.cpp:842] [Rank 20] NCCL watchdog thread started!
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (allen.local, 32927).
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:32927 on [johnson.ece.utexas.edu]:35840.
[I ProcessGroupNCCL.cpp:665] [Rank 18] ProcessGroupNCCL initialized with following options:
NCCL_ASYNC_ERROR_HANDLING: 1
NCCL_DESYNC_DEBUG: 0
NCCL_BLOCKING_WAIT: 0
TIMEOUT(ms): 1800000
USE_HIGH_PRIORITY_STREAM: 0
[I ProcessGroupNCCL.cpp:842] [Rank 18] NCCL watchdog thread started!
| distributed init (world_size 24, global rank 21, local gpu id 5): env://
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (allen.local, 32927).
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:32927 on [johnson.ece.utexas.edu]:35852.
[I socket.cpp:624] [c10d - debug] The client socket will attempt to connect to an IPv6 address of (allen.local, 32927)./ 
[I socket.cpp:787] [c10d] The client socket has connected to [allen.ece.utexas.edu]:32927 on [johnson.ece.utexas.edu]:35858.
[I ProcessGroupNCCL.cpp:665] [Rank 21] ProcessGroupNCCL initialized with following options:
NCCL_ASYNC_ERROR_HANDLING: 1
```

<br>
# Reference:
- [https://leimao.github.io/blog/PyTorch-Distributed-Training/](https://leimao.github.io/blog/PyTorch-Distributed-Training/)
- [https://pytorch.org/tutorials/intermediate/ddp_series_multinode.html](https://pytorch.org/tutorials/intermediate/ddp_series_multinode.html)
- [https://github.com/NVIDIA/nccl/issues/833](https://github.com/NVIDIA/nccl/issues/833)
- [https://github.com/pytorch/pytorch/issues/79388](https://github.com/pytorch/pytorch/issues/79388)
- [https://github.com/pytorch/pytorch/issues/74824#issuecomment-1500144250](https://github.com/pytorch/pytorch/issues/74824#issuecomment-1500144250)
- [https://linuxconfig.org/how-to-enable-disable-firewall-on-ubuntu-20-04-lts-focal-fossa-linux](https://linuxconfig.org/how-to-enable-disable-firewall-on-ubuntu-20-04-lts-focal-fossa-linux)
- [https://askubuntu.com/questions/257263/how-to-display-network-traffic-in-the-terminal](https://askubuntu.com/questions/257263/how-to-display-network-traffic-in-the-terminal)

