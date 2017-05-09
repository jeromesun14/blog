title: Quagga 入门
date: 2017-05-09 16:32:30
toc: true
tags: [quagga, 路由协议]
categories: networks
keywords: [quagga, quickstart, ospf, bgp, rip, isis, 路由, 路由协议, zebra]
description: Quagga 基本使用记录
-------------

* telnet 登陆 zebra CLI，其中通过 `netstat -tlpun | grep zebra` 查看 zebra 开放的 telnet 端口号。


```
root@leaf1:/# netstat -tlpun | grep zebra
tcp        0      0 127.0.0.1:2601          0.0.0.0:*               LISTEN      113/zebra  
root@leaf1:/# telnet localhost 2601
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

Hello, this is Quagga (version 0.99.24.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.


User Access Verification

Password: 
Router> 
Router> 
Router> en
Password: 
Router# 
  clear      Clear stored data
  configure  Configuration from vty interface
  copy       Copy configuration
  debug      Debugging functions (see also 'undebug')
  disable    Turn off privileged mode command
  echo       Echo a message back to the vty
  end        End current mode and change to enable mode.
  exit       Exit current mode and down to previous mode
  help       Description of the interactive help system
  list       Print command list
  logmsg     Send a message to enabled logging destinations
  no         Negate a command or set its defaults
  quit       Exit current mode and down to previous mode
  show       Show running system information
  terminal   Set terminal line parameters
  who        Display who is on vty
  write      Write running configuration to memory, network, or terminal
Router# 
```

* 默认配置放于 `/etc/quagga`，例如本文默认密码为 zebra。

```
root@leaf1:/# cat /etc/quagga/zebra.conf 
hostname Router
password zebra
enable password zebra
!
interface lo
!
line vty
```

本文 bgp 配置：

```
root@leaf1:/# cat /etc/quagga/bgpd.conf 
hostname bgpd
password zebra
enable password zebra
log file /var/log/quagga/bgpd.log

router bgp 64101
  bgp router-id 10.0.1.100
  network 0.0.0.0/0
  network 10.0.1.0/24
  network 10.0.2.0/24
  network 10.1.11.0/24
  network 10.1.12.0/24
  neighbor 10.1.11.2 remote-as 64200
  neighbor 10.1.11.2 timers 1 3
  neighbor 10.1.12.2 remote-as 64200
  neighbor 10.1.12.2 timers 1 3
  maximum-paths 16

access-list all permit any
root@leaf1:/# 
```
