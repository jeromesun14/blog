title: 交换机的三个平面简介——管理平面、控制平面、数据平面
date: 2017-06-16 17:28:00
toc: true
tags: [交换机]
categories: networks
keywords: [computer, network, switch, 交换机, 控制面, 管理面, 数据面, 控制平面, 管理平面, 数据平面, control plane, data plane, management plane]
description: 交换机的三个平面简介。
---

http://www.ruijie.com.cn/special/wzwn/download/pdf/s.pdf

* 数据平面：物理口 <----> 物理口，芯片转发数据。
* 控制平面：物理口 <----> CPU 口，协议报文从物理口 upcall 到 CPU 处理，所有网络协议处理。
* 管理平面：CPU 口 <----> CPU 口，CLI/SNMP/TELNET/WEB/SSH/RMON 等控制方式，配置控制平面相关协议，管理协议，管理交换机设备状态等。

设备架构的不同，导致三个平面的实现方式也不同。

* box
* chassis - fullmesh
* chassis - clos
