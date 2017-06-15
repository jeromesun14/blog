title: Linux虚拟网络设备 TUN/TAP 与 VETH pair 的差异
date: 2017-04-17 17:01:20
toc: true
tags: [tuntap, veth, networks]
categories: networks
keywords: [虚拟网络设备, linux, tuntap, veth pair, tun, tap]
description: Linux 虚拟网络设备 tun/tap 与 veth pair 的差异，及二者常用场景。
---

近来随着 P4 的发展，软件交换机在 ovs 之后又热了起来。软件交换机，顾名思义纯软件实现交换功能，相对于传统交换机的物理端口，软件交换机的端口一般采用虚拟端口的形式。本文探索常用的虚拟网络设备 tuntap 与 veth。

## TUN/TAP

### 基本概念
> A gateway to userspace.

TUN 和 TAP 设备是 Linux 内核虚拟网络设备，纯软件实现。TUN（TUNnel）设备模拟网络层设备，处理三层报文如 IP 报文。TAP 设备模拟链路层设备，处理二层报文，比如以太网帧。TUN 用于路由，而 TAP 用于创建网桥。OS 向连接到 TUN/TAP 设备的用户空间程序发送报文；用户空间程序可像往物理口发送报文那样向 TUN/TAP 口发送报文，在这种情况下，TUN/TAP 设备发送（或注入）报文到 OS 协议栈，就像报文是从物理口收到一样。

> [链接](http://stackoverflow.com/questions/25641630/virtual-networking-devices-in-linux):
> TUN/TAP: The user-space application/VM can read or write an ethernet frame to the tap interface and it would reach the host kernel, where it would be handled like any other ethernet frame that reached the kernel via physical (e.g. eth0) ports. You can potentially add it to a software-bridge (e.g. linux-bridge)

### 如何工作

TUN/TAP 为简单的点对点或以太网设备，不是从物理介质接收数据包，而是从用户空间程序接收；不是通过物理介质发送数据包，而将它们发送到用户空间程序。 
假设您在 tap0 上配置 IPX，那么每当内核向 tap0 发送一个 IPX 数据包时，它将传递给应用程序（例如 VTun）。应用程序加密、压缩数据包，并通过 TCP/UDP 发送到对端。对端的应用程序解压缩、解密接收的数据包，并将数据包写入 TAP 设备，然后内核处理数据包，就像该数据包来自真实的物理设备。

### 用途
用于加密、VPN、隧道、虚拟机等等（encryption, VPN, tunneling,virtual machines）。

### 参考资料

* [kernel doc tuntap](https://www.kernel.org/doc/Documentation/networking/tuntap.txt)
* [virtual networking devices in linux](http://stackoverflow.com/questions/25641630/virtual-networking-devices-in-linux)
* [Linux Networking Explained](http://events.linuxfoundation.org/sites/events/files/slides/2016%20-%20Linux%20Networking%20explained_0.pdf)
* [wikipedia tuntap]

## veth pair

### 基本概念

> Virtual Ethernet Cable
> Bidirectional FIFO
> Often used to cross namespaces

虚拟以太网电缆，注意，Cable，不是 vNIC。使用双向有名管道实现。常用于不同 namespace 之间的通信，即 namespace 数据穿越或容器数据穿越。

由于是 Cable，所以必须以对的形式出现。一个配置样例，[链接](http://www.cnblogs.com/hustcat/p/3928261.html)

> [链接](http://stackoverflow.com/questions/25641630/virtual-networking-devices-in-linux): 
> VETH: Typically used when you are trying to connect two entities which would want to "get hold of" (for lack of better phrase) an interface to forward/receive frames. These entities could be containers/bridges/ovs-switch etc. Say you want to connect a docker/lxc container to OVS. You can create a veth pair and push the first interface to the docker/lxc (say, as a phys interface) and push the other interface to OVS. You cannot do this with TAP.

### 如何工作
向 veth pair 的一端输入数据，veth pair 转换请求发送报文为需要接收处理的报文，将其注入内核协议栈，在另一端能读到此数据。

### 用途
如上所述，常用于不同命名空间之间进行数据穿越。

### 参考资料

* [Linux 上的基础网络设备详解](https://www.ibm.com/developerworks/cn/linux/1310_xiawc_networkdevice/)
* [Linux Networking Explained](http://events.linuxfoundation.org/sites/events/files/slides/2016%20-%20Linux%20Networking%20explained_0.pdf)
