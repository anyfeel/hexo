---
title: Linux 系统优化指令杂记
date: 2018-09-24 11:14:44
tags: Linux
---

## 系统优化的前 5 分钟
[使用如下指令了解机器的整体运行情况](https://medium.com/netflix-techblog/linux-performance-analysis-in-60-000-milliseconds-accc10403c55), [此处有指令的中文解释](https://segmentfault.com/a/1190000004104493)
- `uptime` 检查系统是否宕机重启
- `dmesg | tail`  查看 dmesg 是否有系统层面错误日志
- `vmstat 1` 检查 cpu 的负载情况, 是否存在 cpu 饱和
- `mpstat -P ALL 1` 检查CPU是否存在负载不均衡; 单个过于忙碌的CPU可能意味着整个应用只有单个线程在工作
- `pidstat 1` 观察随时间变化的 cpu、内存等信息
- `iostat -xz 1` 弄清块设备（磁盘）的状况, 包括工作负载和处理性能
- `free -m` 查看剩余内存, 文件和 IO 缓存大小
- `sar -n DEV 1` 检查网络流量的工作负载：rxkB/s和txkB/s, 以及它是否达到限额了
- `sar -n TCP,ETCP 1` 查看系统网络负载, 过多重传可能是服务器过载开始丢包了
- `top` 没什么好多, 大家都会用的

## top 各个指标含义
- Cpu(s)：表示这一行显示CPU总体信息
- 0.0%us：用户态进程占用CPU时间百分比, 不包含renice值为负的任务占用的CPU的时间。
- 0.7%sy：内核占用CPU时间百分比
- 0.0%ni：改变过优先级的进程占用CPU的百分比
- 99.3%id：空闲CPU时间百分比
- 0.0%wa：等待I/O的CPU时间百分比
- 0.0%hi： CPU硬中断时间百分比
- 0.0%si： CPU软中断时间百分比
- 0.0%st:  超线程开启时候, 线程等待另外一个虚拟核完成任务的时间百分比

> note：这里显示数据是所有cpu的平均值, 如果想看每一个cpu的处理情况, 按1即可；折叠, 再次按1

<!-- more --> >

## 中断
### 硬中断
- 硬中断是由硬件产生的，比如，像磁盘，网卡，键盘，时钟等。每个设备或设备集都有它自己的IRQ（中断请求）。基于IRQ，CPU可以将相应的请求分发到对应的硬件驱动上（注：硬件驱动通常是内核中的一个子程序，而不是一个独立的进程）。
- 处理中断的驱动是需要运行在CPU上的，因此，当中断产生的时候，CPU会中断当前正在运行的任务，来处理中断。在有多核心的系统上，一个中断通常只能中断一颗CPU（也有一种特殊的情况，就是在大型主机上是有硬件通道的，它可以在没有主CPU的支持下，可以同时处理多个中断。）。
- 硬中断可以直接中断CPU。它会引起内核中相关的代码被触发。对于那些需要花费一些时间去处理的进程，中断代码本身也可以被其他的硬中断中断。
- 对于时钟中断，内核调度代码会将当前正在运行的进程挂起，从而让其他的进程来运行。它的存在是为了让调度代码（或称为调度器）可以调度多任务。

### 软中断
- 软中断的处理非常像硬中断。然而，它们仅仅是由当前正在运行的进程所产生的。
- 通常，软中断是一些对I/O的请求。这些请求会调用内核中可以调度I/O发生的程序。对于某些设备，I/O请求需要被立即处理，而磁盘I/O请求通常可以排队并且可以稍后处理。根据I/O模型的不同，进程或许会被挂起直到I/O完成，此时内核调度器就会选择另一个进程去运行。I/O可以在进程之间产生并且调度过程通常和磁盘I/O的方式是相同。
- 软中断仅与内核相联系。而内核主要负责对需要运行的任何其他的进程进行调度。一些内核允许设备驱动的一些部分存在于用户空间，并且当需要的时候内核也会调度这个进程去运行。
-  软中断并不会直接中断CPU。也只有当前正在运行的代码（或进程）才会产生软中断。这种中断是一种需要内核为正在运行的进程去做一些事情（通常为I/O）的请求。有一个特殊的软中断是Yield调用，它的作用是请求内核调度器去查看是否有一些其他的进程可以运行。


### 释疑
> 对于软中断，I/O 操作是否是由内核中的 I/O 设备驱动程序完成？

对于 I/O 请求，内核会将这项工作分派给合适的内核驱动程序，这个程序会对 I/O 进行队列化，以可以稍后处理（通常是磁盘I/O），或如果可能可以立即执行它。通常，当对硬中断进行回应的时候，这个队列会被驱动所处理。当一个 I/O 请求完成的时候，下一个在队列中的 I/O 请求就会发送到这个设备上。

> 软中断所经过的操作流程是比硬中断的少吗？换句话说，对于软中断就是：进程 -> 内核中的设备驱动程序；对于硬中断：硬件 -> CPU 内核中的设备驱动程序？

是的，软中断比硬中断少了一个硬件发送信号的步骤。产生软中断的进程一定是当前正在运行的进程，因此它们不会中断CPU。但是它们会中断调用代码的流程。

如果硬件需要 CPU 去做一些事情，那么这个硬件会使 CPU 中断当前正在运行的代码。而后 CPU 会将当前正在运行进程的当前状态放到堆栈（stack）中，以至于之后可以返回继续运行。这种中断可以停止一个正在运行的进程；可以停止正处理另一个中断的内核代码；或者可以停止空闲进程。

## 查看 cpu 超线程和 cpu 物理核对应关系
```
~$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              2
On-line CPU(s) list: 0,1
Thread(s) per core:  1
Core(s) per socket:  2
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               61
Model name:          Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz
Stepping:            4
CPU MHz:             2699.998
BogoMIPS:            5399.99
Hypervisor vendor:   KVM
Virtualization type: full
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            3072K
NUMA node0 CPU(s):   0,1

~$ cat /sys/devices/system/node/node1/cpulist
1,3,5,7,9,11,13,15,17,19,21,23,25,27,29,31,33,35,37,39

~$ cat /sys/devices/system/node/node0/cpulist
0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38
```
机器总共有 2 个物理核心，每个物理核心有 10 个逻辑核，每个逻辑核有 2 个超线程

## 查看进程和 cpu 亲缘性绑定关系
```
ps -eo pid,argssr
pid  - 进程ID
args - 该进程执行时传入的命令行参数
psr  - 分配给进程的逻辑CPU

~$ ps -eo pid,argssr | grep nginx
9073 nginx: master process /usr/   1
9074 nginx: worker process         0
9075 nginx: worker process         1
9076 nginx: worker process         2
9077 nginx: worker process         3
```

## 查看中断号和 cpu 绑定关系
```
~$ cat /proc/interrupts

            CPU0       CPU1
 162: 1051061576          0  IR-PCI-MSI-edge      eth0-TxRx-0
 163:   85025461  133849653  IR-PCI-MSI-edge      eth0-TxRx-1
 164:   88387458          0  IR-PCI-MSI-edge      eth0-TxRx-2
 165:  138497867          0  IR-PCI-MSI-edge      eth0-TxRx-3
 166:  187011870          0  IR-PCI-MSI-edge      eth0-TxRx-4
 167:   71803161   35003925  IR-PCI-MSI-edge      eth0-TxRx-5
 168:   65314317          0  IR-PCI-MSI-edge      eth0-TxRx-6
 169:   64069078          0  IR-PCI-MSI-edge      eth0-TxRx-7
 170:  138158880          0  IR-PCI-MSI-edge      eth0-TxRx-8
 171:   40424137   20524266  IR-PCI-MSI-edge      eth0-TxRx-9
 172:  134531725          0  IR-PCI-MSI-edge      eth0-TxRx-10

~$ sudo cat /proc/irq/165/smp_affinity
00000000,00000000,00000000,00000000,00000000,00000008
```
首先查看系统所有中断号，查看多对列网卡 eth0 的中断号
查看中断号分配的 cpu, 0x00000008 代表 cpu 8

## others
```
ethtools -S eth0 查看网络的队列信息
ethanol -i eth0 查看网卡信息
lspci -vvv 查看pci 相关信息

```

## iperf 网卡测试
`-P` 指定一个并发, 如果指定多个并发，统计信息不友好
```
server:
iperf -s

client:
iperf -c 10.200.136.41 -p 5001  -P 1 -t 60 -i 1
```

## 查看网卡情况
netstat 查看 Errors
ifconfig 查看 dropped 数量
```
~$ watch 'netstat -s --udp'
Udp:
    437.0k/s packets received
    0.0/s packets to unknown port received.
    386.9k/s packet receive errors
    0.0/s packets sent
    RcvbufErrors:  123.8k/s
    SndbufErrors: 0
    InCsumErrors: 0

~$ ifconfig
enp0s8:
    flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    inet 192.168.56.101  netmask 255.255.255.0  broadcast 192.168.56.255
    inet6 fe80::b5e6:56d6:48dd:e317  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:e6:c8:ac  txqueuelen 1000  (Ethernet)
    RX packets 71  bytes 20634 (20.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
    TX packets 81  bytes 13593 (13.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
