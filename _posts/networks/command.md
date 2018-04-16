title: 网络相关命令入门
date: 2018-04-12 18:18:0
toc: true
tags: [network]
categories: networks
keywords: [arp, route, tcpdump, -n, slow, 很慢, command, 命令]
description: 网络相关命令入门使用
---

## FAQ
### 命令执行很慢 / 报文没显示，但是已抓到 / 报文抓得不全 / ...

Linux 网络相关命令，如果执行起来很慢，尝试一下 `-n` 选项。没了 ip 地址、L4 port 解析成名称，速度杠杠的。

> -n     Don't convert addresses (i.e., host addresses, port numbers, etc.) to names.

```
admin@sonic:~$ sudo tcpdump -i Ethernet52
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on Ethernet52, link-type EN10MB (Ethernet), capture size 262144 bytes
^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C





^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^C^Z
[1]+  Stopped                 sudo tcpdump -i Ethernet52
admin@sonic:~$
admin@sonic:~$ sudo tcpdump -i Ethernet52 -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on Ethernet52, link-type EN10MB (Ethernet), capture size 262144 bytes
08:07:35.175472 IP 20.0.0.2 > 20.0.0.1: ICMP echo request, id 32684, seq 89, length 64
08:07:35.175557 IP 20.0.0.1 > 20.0.0.2: ICMP echo reply, id 32684, seq 89, length 64
08:07:36.175431 IP 20.0.0.2 > 20.0.0.1: ICMP echo request, id 32684, seq 90, length 64
08:07:36.175520 IP 20.0.0.1 > 20.0.0.2: ICMP echo reply, id 32684, seq 90, length 64
08:07:37.175374 IP 20.0.0.2 > 20.0.0.1: ICMP echo request, id 32684, seq 91, length 64
08:07:37.175463 IP 20.0.0.1 > 20.0.0.2: ICMP echo reply, id 32684, seq 91, length 64
08:07:38.175297 IP 20.0.0.2 > 20.0.0.1: ICMP echo request, id 32684, seq 92, length 64
08:07:38.175377 IP 20.0.0.1 > 20.0.0.2: ICMP echo reply, id 32684, seq 92, length 64
08:07:39.175221 IP 20.0.0.2 > 20.0.0.1: ICMP echo request, id 32684, seq 93, length 64
08:07:39.175317 IP 20.0.0.1 > 20.0.0.2: ICMP echo reply, id 32684, seq 93, length 64
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

## tcpdump 如何看 tag?

tcpdump 带 `-e` 选项。

```
 -e     Print the link-level header on each dump line.  This can be used, for example, to print MAC layer addresses for protocols such as Ethernet and IEEE 802.11.
```
