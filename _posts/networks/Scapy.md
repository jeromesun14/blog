title: Scapy简述
date: 2015-09-22 20:00:30
toc: true
tags: [网络测试]
categories: networks
keywords: [发包, scapy, 网络测试, send packet, test, network, python,]
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


## 常用操作
### 发送pcap格式的报文

1. 读取pcap文件
2. 以二层的形式发包。

```
from scapy.all import *
buffer=rdpcap("/home/sunyongfeng/test/1029.pcap")
sendp(buffer, iface="eth2",inter=0.001, loop=1, count=1000000000000)
sendp(ETHER()/IP(dst="1.1.1.1")/TCP()/"You are offline.", iface="eth2",inter=0.001, loop=1, count=1000000000000)
```

### 发送 raw 报文

使用 `Raw` 类。

```
$ sudo scapy 
INFO: Can't import python gnuplot wrapper . Won't be able to plot.
INFO: Can't import PyX. Won't be able to use psdump() or pdfdump().
WARNING: No route found for IPv6 destination :: (no default route?)
INFO: Can't import python Crypto lib. Won't be able to decrypt WEP.
INFO: Can't import python Crypto lib. Disabled certificate manipulation tools
Welcome to Scapy (2.2.0)
>>> pkt = Raw('\x00\xe0\xec\x5a\x68\x34\x00\xe0\xec\x83\xb4\x78\x08\x00\x45\xc0\x00\x3c\x38\xe7\x40\x00\x01\x06\x18\x13\x14\x00\x00\x01\x14\x00\x00\x02\x8d\x49\x00\xb3\xe6\x7e\x5f\xfa\x00\x00\x00\x00\xa0\x02\x72\x10\x53\x43\x00\x00\x02\x04\x05\xb4\x04\x02\x08\x0a\x00\xc4\x85\x70\x00\x00\x00\x00\x01\x03\x03\x07')
>>> sendp(pkt, iface="Ethernet52",  inter=1, loop=1, count=10) 
..........
Sent 10 packets.
```

### 发送特定长度报文

使用 `RandString(size=xxx)`。

```
>>> a=IP(dst="76.200.0.2",ttl=1)/TCP(sport=176)/Raw(RandString(size=8000))   
>>> send(a, iface="Ethernet76",  inter=0.001, loop=1, count=1000)  
```

### 使用 send 发送三层报文提示 MAC 未解析

使用 `send` 发送三层报文的时候，Ethernet 的信息由 PF_PACKET 自行解析，不需要提供，否则会提示 `WARNING: Mac address to reach destination not found. Using broadcast.`。

```
$ sudo scapy3k          
WARNING: No route found for IPv6 destination :: (no default route?). This affects only IPv6
INFO: Please, report issues to https://github.com/phaethon/scapy
INFO: Can't import python cryptography lib. Won't be able to decrypt WEP.
INFO: Can't import python cryptography lib. Disabled IPsec encryption/authentication.
INFO: Can't import python cryptography. Disabled certificate manipulation tools
WARNING: IPython not available. Using standard Python shell instead.
Welcome to Scapy (3.0.0)
>>> a=Ether(dst="00:e0:ec:b1:b6:92")/IP(dst="76.200.0.2",ttl=2)/TCP()                           
>>> send(a, iface="Ethernet76", inter=0.001, loop=1, count=1000)
WARNING: Mac address to reach destination not found. Using broadcast.
.WARNING: Mac address to reach destination not found. Using broadcast.
.WARNING: more Mac address to reach destination not found. Using broadcast.
..........................................................................................................................................^C
Sent 141 packets.
>>> a=IP(dst="76.200.0.2",ttl=2)/TCP()                                     
>>> send(a, iface="Ethernet76")                                      
.
Sent 1 packets.
>>> send(a, iface="Ethernet76",  inter=0.001, loop=1, count=1000)
........................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
Sent 1000 packets.
>>>
```

### 发送 ARP 报文

op 还有 `is-at`。

```
from scapy.all import *
sendp(Ether(src="00:00:00:00:00:AA", dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.3.4", psrc="192.168.3.1",op="who-has"), iface="enp132s0f1", inter=0.1, loop=1, count=1) 
```

### 抓包并基于该包构造、发送新包

以下样例抓特定接口 ARP 请求报文，并返回 ARP 响应报文，类似 proxy ARP 功能。

```
from scapy.all import *
import threading

def fakeResponse(packet):
    for pkt in packet:
        smac = pkt[Ether].src
        sip = pkt[ARP].psrc
        tip = pkt[ARP].pdst
        hwsrc = pkt[ARP].hwsrc
        # 1 REQUEST, 2 REPLY
        op  = pkt[ARP].op
        
        if op != 1:
            return
        
        #print "### RECV PACKET START ###"
        #pkt.show()
        #print "### RECV PACKET END ###"

        reply = Ether(dst=smac)/ARP(op="is-at", psrc=tip, pdst=sip, hwdst=hwsrc)/Raw(RandString(size=48))
        #print "### SEND PACKET START ###"
        #reply.show()
        #print "### SEND PACKET END ###"
        global lock
        global pkts
        global thd
        pkts_tmp = []
        with lock:
            pkts.append(reply)
            if len(pkts) >= thd:
                pkts_tmp = copy.copy(pkts)
                pkts = []
        if len(pkts_tmp) != 0:
            sendp(pkts_tmp, iface='eth4', verbose=0, inter=0.005)

def timer_handler():
    global lock
    global pkts
    global thd
    pkts_tmp = []
    with lock:
        pkts_tmp = copy.copy(pkts)
        pkts = []
    if len(pkts_tmp) != 0:
        sendp(pkts_tmp, iface='eth4', verbose=0, inter=0.005)
    
    global timer
    timer = threading.Timer(0.5, timer_handler)
    timer.start()

pkts = []
thd = 50
lock = threading.Lock()
timer = threading.Timer(0.5, timer_handler)
timer.start()
sniff(iface='eth4', filter="arp", prn=fakeResponse)
```

## 常见问题

### 发包慢

* 一个一个发包，比按 pkts list 发包慢很多，原因为一个一个发，会一直在 bind socket。


