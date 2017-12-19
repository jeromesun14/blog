title: Linux内核源码处理ICMP packet too big与DF
date: 2017-12-19 09:52:24
toc: true
tags: [Linux, packet]
categories: c
keywords: [Linux, kernel, don't fragment, DF, 分片]
description: Linux 内核源码如何处理 ICMP packet too big 和 don't fragment。
---

* RFC 1191
* man 7 ip

结论
------

* ICMP packet too big
对ICMP packet too big的处理，TCP/UDP最终都对对应路由条目进行MTU更新。而TCP的MTU、MSS、窗口大小和最终段大小的关系比较复杂，目前没有看明白。
* Don't fragment
DF位的默认配置，受系统配置控制，Ubuntu 14.04默认开启MTU发现功能。推断结果：UDP大报文默认未置上，而TCP大报文默认置上。

Linux内核如何处理ICMP Packet too big
------------------------------------------------

### ICMP流程
报文类型：
```
23 #define ICMP_DEST_UNREACH       3       /* Destination Unreachable      */
43 #define ICMP_FRAG_NEEDED        4       /* Fragmentation Needed/DF set  */
```

[icmp_rcv(struct sk_buff *skb)](http://lxr.free-electrons.com/source/net/ipv4/icmp.c?v=3.10#L848)，根据ICMP类型处理skb：
```
icmp_pointers[icmph->type].handler(skb);
```
[icmp_pointers](http://lxr.free-electrons.com/source/net/ipv4/icmp.c?v=3.10#L965)的定义显示，ICMP_DEST_UNRAECH的handler为[icmp_unreach](http://lxr.free-electrons.com/source/net/ipv4/icmp.c?v=3.10#L663)，后者获取出ICMP头部的mtu后，投递给[icmp_socket_deliver(skb, mtu)](http://lxr.free-electrons.com/source/net/ipv4/icmp.c?v=3.10#L638)。
`icmp_socket_deliver`最终将skb和mtu值投递给` ipprot->err_handler(skb, info);`


### TCP流程
[tcp_v4_err(skb, info)](http://lxr.free-electrons.com/source/net/ipv4/tcp_ipv4.c?v=3.10#L326)，
调用[tcp_v4_mtu_reduced(struct sock *sk)](http://lxr.free-electrons.com/source/net/ipv4/tcp_ipv4.c?v=3.10#L271)

更新路由的MTU，[inet_csk_update_pmtu(struct sock *sk, u32 mtu)](http://lxr.free-electrons.com/source/net/ipv4/inet_connection_sock.c?v=3.10#L929)
找到路由`inet_csk_rebuild_route(sk, &inet->cork.fl);`
更新路由MTU`dst->ops->update_pmtu(dst, sk, NULL, mtu);`

`tcp_v4_err`，如果MTU变小，且可分片，则会[tcp_sync_mss(struct sock *sk, u32 pmtu)](http://lxr.free-electrons.com/source/net/ipv4/tcp_output.c?v=3.10#L1296)。关于TCP的MSS与MTU相关的内容很多：

```
1274 /* This function synchronize snd mss to current pmtu/exthdr set.
1275 
1276    tp->rx_opt.user_mss is mss set by user by TCP_MAXSEG. It does NOT counts
1277    for TCP options, but includes only bare TCP header.
1278 
1279    tp->rx_opt.mss_clamp is mss negotiated at connection setup.
1280    It is minimum of user_mss and mss received with SYN.
1281    It also does not include TCP options.
1282 
1283    inet_csk(sk)->icsk_pmtu_cookie is last pmtu, seen by this function.
1284 
1285    tp->mss_cache is current effective sending mss, including
1286    all tcp options except for SACKs. It is evaluated,
1287    taking into account current pmtu, but never exceeds
1288    tp->rx_opt.mss_clamp.
1289 
1290    NOTE1. rfc1122 clearly states that advertised MSS
1291    DOES NOT include either tcp or ip options.
1292 
1293    NOTE2. inet_csk(sk)->icsk_pmtu_cookie and tp->mss_cache
1294    are READ ONLY outside this function.         --ANK (980731)
1295  */
```

### 处理TCP MSS
未看懂。

### UDP流程
[__udp4_lib_err(struct sk_buff *skb, u32 info, struct udp_table *udptable)](http://lxr.free-electrons.com/source/net/ipv4/udp.c?v=3.10#L608)
[ipv4_sk_update_pmtu(struct sk_buff *skb, struct sock *sk, u32 mtu)](http://lxr.free-electrons.com/source/net/ipv4/route.c?v=3.10#L983)，也是基于路由更新MTU。
[__ip_rt_update_pmtu](http://lxr.free-electrons.com/source/net/ipv4/route.c?v=3.10#L911)

ping报文的处理也是类似，见[ping_err](http://lxr.free-electrons.com/source/net/ipv4/ping.c?v=3.10#L369)。

Linux内核对DF的默认处理
---------------------------------

经阅读代码，发现控制DF标志的地方主要有两处：
1. inet_sock.pmtudisc
2. sk_buff.local_df

以关键字pmtudisc和local_df做全局字符串搜索，发现对pmtudisc和local_df有赋值的地方很少。如下各小节分析。

另附：[IP MTU DISCOVER相关宏](http://lxr.free-electrons.com/source/include/uapi/linux/in.h?v=3.10#L91)
```
 90 /* IP_MTU_DISCOVER values */
 91 #define IP_PMTUDISC_DONT                0       /* Never send DF frames */
 92 #define IP_PMTUDISC_WANT                1       /* Use per route hints  */
 93 #define IP_PMTUDISC_DO                  2       /* Always DF            */
 94 #define IP_PMTUDISC_PROBE               3       /* Ignore dst pmtu      */
```

### IP报文发送出口
源代码：[ip_queue_xmit](http://lxr.free-electrons.com/source/net/ipv4/ip_output.c?v=3.10#L326)和[__ip_make_skb](http://lxr.free-electrons.com/source/net/ipv4/ip_output.c?v=3.10#L1314)，后者用于UDP。

```
382         if (ip_dont_fragment(sk, &rt->dst) && !skb->local_df)
383                 iph->frag_off = htons(IP_DF);
384         else
385                 iph->frag_off = 0;

248 int ip_dont_fragment(struct sock *sk, struct dst_entry *dst)
249 {
250         return  inet_sk(sk)->pmtudisc == IP_PMTUDISC_DO ||
251                 (inet_sk(sk)->pmtudisc == IP_PMTUDISC_WANT &&
252                  !(dst_metric_locked(dst, RTAX_MTU)));
253 }
```

### AF_INET协议族
[inet_create](http://lxr.free-electrons.com/source/net/ipv4/af_inet.c?v=3.10#L375)，依赖`ipv4_config`配置。该配置体现在`/proc/sys/net/ipv4/ip_no_pmtu_disc`，目前Ubuntu 14.04上默认`ip_no_pmtu_disc=0`，即由每个路由确定。

```
374         if (ipv4_config.no_pmtu_disc)
375                 inet->pmtudisc = IP_PMTUDISC_DONT;
376         else
377                 inet->pmtudisc = IP_PMTUDISC_WANT;
```

### 系统配置
可用`sysctl`命令配置，或通过`/etc/sysctl.conf`配置文件配置`ip_no_pmtu_disc`值，最终体现在`/proc/sys/net/ipv4/ip_no_pmtu_disc`。[源码链接](http://lxr.free-electrons.com/source/net/ipv4/sysctl_net_ipv4.c?v=3.10#L316)。

### ICMP报文
默认创建ICMP sk时，不发送DF报文。[icmp_sk_init(struct net *net)](http://lxr.free-electrons.com/source/net/ipv4/icmp.c?v=3.10#L1074)。

PS：ICMP报文不超过576字节的限制在[icmp_send](http://lxr.free-electrons.com/source/net/ipv4/icmp.c?v=3.10#L481)接口中有体现。
