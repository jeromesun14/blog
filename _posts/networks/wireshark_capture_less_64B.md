title: 为什么 wireshark 可以抓到小于 64 字节的报文？
date: 2017-02-16 10:39:18
toc: false
tags: [网络测试]
categories: networks
keywords: [computer networks, wireshark, 抓包, capture]
description: wireshark 可以抓到小于 64 字节报文的原因。
---

底层不提供显示 FCS
-------
以太网报文大小：

* Preamble, 8B
* Destination MAC address, 6B
* Source MAC address, 6B
* Type/Length, 2B
* User Data, 46 - 1500B
* Frame Check Sequence (FCS), 4B

前导码一般被物理层处理掉，且底层一般在驱动层剥除 FCS，因此一般情况下，Wireshark 抓到的最小包为 6B + 6B + 2B + 46B = 60B。

> [链接](https://wiki.wireshark.org/Ethernet)：
>
> As the Ethernet hardware filters the preamble, it is not given to Wireshark or any other application. Most Ethernet interfaces also either don't supply the FCS to Wireshark or other applications, or aren't configured by their driver to do so; therefore, Wireshark will typically only be given the green fields, although on some platforms, with some interfaces, the FCS will be supplied on incoming packets.
> 
> Ethernet packets with less than the minimum 64 bytes for an Ethernet packet (header + user data + FCS) are padded to 64 bytes, which means that if there's less than 64-(14+4) = 46 bytes of user data, extra padding data is added to the packet.
> 
> Beware: the minimum Ethernet packet size is commonly mentioned at 64 bytes, which is including the FCS. This can be confusing as the FCS is often not shown by Wireshark, simply because the underlying mechanisms simply don't supply it.


wireshark 抓到小于 60 字节的报文？
------------------
* 首先 wireshark 抓到小于 60 字节报文都是本机发出的。
* 其次 wireshark 能抓到本机小于 60 字节报文的原因：填充报文到 60 字节的行为在 wireshark 抓包后。

> [链接](https://ask.wireshark.org/questions/1846/wireshark-capture-of-ethernet-frame-size-shows-as-43-bytes)，What you see is 14 bytes ethernet header, 20 bytes IP header, 8 bytes ICMP header, 1 byte payload, equals 43. The NIC will later add padding bytes to get it up to 60 bytes and adds the FCS. Voila, 64 bytes - but Wireshark grabs the packet too early as Jaap already explained, so you see 43 bytes on the sender side.
