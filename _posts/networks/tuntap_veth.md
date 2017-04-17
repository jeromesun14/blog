title: Linux虚拟网络设备 TUN/TAP 与 VETH pair 的差异
date: 2017-04-17 17:01:20
toc: true
tags: [tuntap, veth, networks]
categories: networks
keywords: [虚拟网络设备, linux, tuntap, veth pair, tun, tap]
description: Linux 虚拟网络设备 tun/tap 与 veth pair 的差异，及二者常用场景。
---

## TUN/TAP

TUN/TAP 设备

> [wikipedia]():
> In computer networking, TUN and TAP are virtual network kernel devices. Being network devices supported entirely in software, they differ from ordinary network devices which are backed up by hardware network adapters.
> 
> TUN (namely network TUNnel) simulates a network layer device and it operates with layer 3 packets like IP packets. TAP (namely network tap) simulates a link layer device and it operates with layer 2 packets like Ethernet frames. TUN is used with routing, while TAP is used for creating a network bridge.
> 
> Packets sent by an operating system via a TUN/TAP device are delivered to a user-space program which attaches itself to the device. A user-space program may also pass packets into a TUN/TAP device. In this case the TUN/TAP device delivers (or "injects") these packets to the operating-system network stack thus emulating their reception from an external source.

> [链接](http://stackoverflow.com/questions/25641630/virtual-networking-devices-in-linux):
> TUN/TAP: The user-space application/VM can read or write an ethernet frame to the tap interface and it would reach the host kernel, where it would be handled like any other ethernet frame that reached the kernel via physical (e.g. eth0) ports. You can potentially add it to a software-bridge (e.g. linux-bridge)

## veth pair

必须以对的形式出现，用于 namespace 之间的通信，例如 docker 间。

> [链接](http://stackoverflow.com/questions/25641630/virtual-networking-devices-in-linux): 
> VETH: Typically used when you are trying to connect two entities which would want to "get hold of" (for lack of better phrase) an interface to forward/receive frames. These entities could be containers/bridges/ovs-switch etc. Say you want to connect a docker/lxc container to OVS. You can create a veth pair and push the first interface to the docker/lxc (say, as a phys interface) and push the other interface to OVS. You cannot do this with TAP.

