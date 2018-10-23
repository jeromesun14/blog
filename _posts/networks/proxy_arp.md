title: ARP 代理
date: 2018-10-23 08:43:11
toc: true
tags: [ARP, networks]
categories: networks
keywords: [computer, networks, arp, proxy arp, 代理]
description: ARP 代理
---

## Linux 配置 ARP 代理

```
bridge# echo 1 > /proc/sys/net/ipv4/conf/eth4/proxy_arp
bridge# echo 1 > /proc/sys/net/ipv4/conf/eth4/proxy_arp_pvlan
bridge# echo 1 > /proc/sys/net/ipv4/ip_forward
```

## 参考材料

* http://linux-ip.net/html/ether-arp-proxy.html
* https://wiki.debian.org/BridgeNetworkConnectionsProxyArp
