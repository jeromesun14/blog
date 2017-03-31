title: vagrant 使用记录
date: 2017-03-31 11:39:24
toc: true
tags: [vagrant, virtualBox]
categories: programmer
keywords: [vagrant, virtualBox, host ping virtualBox]
description: Vagrant + virtualBox 使用记录
---

# vagrant 如何 ping 通虚拟机

## vagrant 默认网络通信机制
vagrant 默认使用 NAT 端口映射，可以通过端口映射访问虚机内部端口号。但是默认情况下，vagrant 可以访问外网，但是外部世界访问不了 vagrant （类似以前在学校，学校内可以访问外网，但是外网访问不了教育网）。
如果你想在外部访问 vagrant 虚机提供的服务，需要配置端口映射，比如将虚机的端口 80 映射成 host 机的端口 8080，详见 [vagrant networking basic usage](https://www.vagrantup.com/docs/networking/basic_usage.html)：

```vagrantfile
  config.vm.network "forwarded_port", guest: 80, host: 8080
```

默认情况下，vagrant 虚机中有一张网卡，默认IP `10.0.2.15`。虚机可上外网，host 无法 ping 通虚机。

## vagrant 其他通信机制

### private network
详见[vagrant networking private_network](https://www.vagrantup.com/docs/networking/private_network.html)。

通常利用 provider 创建的虚拟网卡，例如 virtualBox 的 Host-Only Network。

```
以太网适配器 VirtualBox Host-Only Network:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::a801:a2e7:cbd9:7126%3
   IPv4 地址 . . . . . . . . . . . . : 192.168.56.1
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . :
```

vagrant 的配置：

* 静态IP：

```
  config.vm.network "private_network", ip: "192.168.56.10"
```

* 动态IP：

```
  config.vm.network "private_network", type: "dhcp"
```

配置后虚机中新增一张网卡，如下的 enp0s8。

```
vagrant@ubuntu-xenial:~$ sudo ifconfig -a
enp0s3    Link encap:Ethernet  HWaddr 02:6a:71:da:1d:13
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::6a:71ff:feda:1d13/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:45731 errors:0 dropped:0 overruns:0 frame:0
          TX packets:20045 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:37231206 (37.2 MB)  TX bytes:1387674 (1.3 MB)

enp0s8    Link encap:Ethernet  HWaddr 08:00:27:c4:aa:2c
          inet addr:192.168.56.10  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fec4:aa2c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:818 errors:0 dropped:0 overruns:0 frame:0
          TX packets:575 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:96862 (96.8 KB)  TX bytes:100624 (100.6 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:480 (480.0 B)  TX bytes:480 (480.0 B)
```

此是 host 可以 ping 通虚机。

```
C:\Users\R0>ping 192.168.56.10

正在 Ping 192.168.56.10 具有 32 字节的数据:
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64

192.168.56.10 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 0ms，最长 = 0ms，平均 = 0ms

C:\Users\R0>
```

### public network
利用 provider 提供的 bridge （桥接功能）。

