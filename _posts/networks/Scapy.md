title: Scapy简述
date: 2015-09-22 20:00:30
toc: true
tags: [网络测试]
categories: networks
keywords: [发包, scapy, 网络测试]
description: Python发包工具scapy使用简介，待进一步补充
---

scapy常用接口
------------------

### get_if_list() 获取接口列表
支持自动被全函数集的功能。
```
>>> get_if_list()
['lo', 'eth0', 'eth1', 'eth2']
```

### help() 获取帮助
```
>>> help(sendp)
Help on function sendp in module scapy.sendrecv:

sendp(x, inter=0, loop=0, iface=None, iface_hint=None, count=None, verbose=None, realtime=None, *args, **kargs)
    Send packets at layer 2
    sendp(packets, [inter=0], [loop=0], [verbose=conf.verb]) -> None
(END)
```

<!--more-->

### 发送/接收接口
1. 只发不收
	* send()，三层发包
	* sendp()，二层发包
2. 发且收
	* sr()，sr1()， srloop()，三层发包
	* srp()，srp1()，srploop，二层发包

> The send and receive functions family will not only send stimuli and sniff responses but also match sent stimuli with received responses. 
> 
> Function sr(),sr1(),and srloop() all work at layer 3.
> 
> Sr() is for sending and receiving packets,it returns a couple of packet and answers,and the unanswered packets.but sr1() only receive the first answer.srloop() is for sending a packet in loop and print the answer each time


发送pcap格式的报文
--------------------------
1. 读取pcap文件
2. 以二层的形式发包。

```
from scapy.all import *
buffer=rdpcap("/home/sunyongfeng/test/1029.pcap")
sendp(buffer, iface="eth2",inter=0.001, loop=1, count=1000000000000)
```

