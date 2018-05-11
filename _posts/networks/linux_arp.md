title: linux arp
date: 2018-05-11 16:48:30
toc: true
tags: [arp, networks]
categories: networks
keywords: [kernel, linux, debian, arp]
description: linux arp 学习笔记。
---

详见：

* cumulus linux 文档：https://support.cumulusnetworks.com/hc/en-us/articles/202012933-Changing-ARP-timers-in-Cumulus-Linux
* arp man 文档：https://linux.die.net/man/7/arp

Linux ARP 的配置基于端口，但是有默认配置，位置详见 `/proc/sys/net/ipv4/neigh/default/*`

## ARP 老化时间

默认 base_reachable_time 为 1800 秒，即老化时间为 900s 到 2700s 的随机值（15分钟-45分钟）。

ARP 老化时间由 base_reachable_time 决定，当内核软转没有使用到这个 ARP 一段时间后（between base_reachable_time/2 and 3*base_reachable_time/2），将这个 ARP 从 reachable 状态置为 stale 状态。后由 gc（garbage collector）回收。

判断 ARP 是否为 stale 状态的周期为 gc_stale_time（默认 60 秒），gc 运行的间隔为 gc_interval（默认 30 秒）。

> * base_reachable_time (since Linux 2.2)
> 
> Once a neighbor has been found, the entry is considered to be valid for at least a random value between **base_reachable_time/2 and 3*base_reachable_time/2**. An entry's validity will be extended if it receives positive feedback from higher level protocols. Defaults to 30 seconds. This file is now obsolete in favor of base_reachable_time_ms.
> 
> * base_reachable_time_ms (since Linux 2.6.12)
> 
> As for base_reachable_time, but measures time in milliseconds. Defaults to 30000 milliseconds.

## ARP 容量

目前默认配置为 512。

* gc_thresh1，如果 ARP cache 小于这个数，gc 不会运行。
* gc_thresh2，ARP cache 的 soft 最大值。gc 允许 ARP cache 超过这个数 5 秒，5 秒后运行 gc。
* gc_thresh3，ARP cache 的 hard 最大值。一旦 ARP cache 超过这个数，gc 会一直运行。

> * gc_thresh1 (since Linux 2.2)
> 
> The minimum number of entries to keep in the ARP cache. The garbage collector will not run if there are fewer than this number of entries in the cache. Defaults to 128.
>
> * gc_thresh2 (since Linux 2.2)
> 
> The soft maximum number of entries to keep in the ARP cache. The garbage collector will allow the number of entries to exceed this for 5 seconds before collection will be performed. Defaults to 512.
>
> * gc_thresh3 (since Linux 2.2)
> 
> The hard maximum number of entries to keep in the ARP cache. The garbage collector will always run if there are more than this number of entries in the cache. Defaults to 1024.

