title: MAC地址
date: 2015-07-02 22:07:18
toc: true
tags: [L2]
categories: networks
keywords: [computer networks, mac, layer2]
description: MAC地址概述
---

概述
-----
**MAC Address**，Media Access Control Address，亦称为EHA（Ethernet Hardware Address）、硬件地址、物理地址（Physical Address）。在OSI节层模型中，属于第二层链路层概念。一个MAC地址唯一指定一台设备，全球唯一，并且通常烧写在固件中。

MAC地址由IEEE（Institute of Electrical and Electronics Engineers）定义，有三种：MAC-48、EUI-48、EUI-64。EUI， Extended Unique Identifier。

MAC-48，地址空间48比特，支持2^48即281,474,976,710,656个MAC地址。MAC-48的设计目标是一百年内够用。常以16进制数表示MAC地址，通用表示方法有三种格式，如`01-23-45-67-89-ab`、`01:23:45:67:89:ab`和`0123.4567.89ab`。

<!--more-->

组成
------
MAC地址如下图所示，其前3字节表示OUI（Organizationally Unique Identifier），由IEEE的注册管理机构给不同厂家分配的代码，区分不同的厂家，[附IEEE当前OUI分配](http://standards-oui.ieee.org/oui.txt)。后3字节由厂家自行分配。

MAC地址最高字节（MSB）的低第二位（LSb）表示这个MAC地址是全局的还是本地的，即U/L（Universal/Local）位，如果为0，表示是全局地址。所有的OUI这一位都是0。

MAC地址最高字节（MSB）的低第一位(LSb），表示这个MAC地址是单播还是多播。0表示单播。

![MAC地址](https://upload.wikimedia.org/wikipedia/commons/thumb/9/94/MAC-48_Address.svg/500px-MAC-48_Address.svg.png)

MAC-48、EUI-48与EUI-64的关系
---------------------------------
疑问：
* EUI-48 MA-L、MA-M、MA-S有没有可能冲突？
* EUI-48的MA-M/MA-S的应用？

### EUI-48 ###
IEEE的《[Guidelines for 48-Bit Global Identifier (EUI-48)](https://standards.ieee.org/develop/regauth/tut/eui48.pdf)》对EUI-48标识符的使用范围进行定义： 

* 用为IEEE 802或类IEEE 802网络设备的硬件地址。
* 用为特定硬件设备的标识，不需要为网络设备。

EUI-48有三种格式：

* 前24位由IEEE Registration Authority (IEEE RA) 分配，后24位由MA-L（MAC Address Block Large）的厂商或组织分配。这种格式即MAC-48。
* 前28位由IEEE RA分配，后20位由MA-M（MAC Address Block Medium）的厂商或组织分配。
* 前36位由IEEE RA分配，后12位由MA-S（MAC Address Block Small）分配。

### EUI-64###
待整理

应用
-----
（来自wikipedia）
###MAC-48###
* Ethernet
* 802.11 wireless networks
* Bluetooth
* IEEE 802.5 token ring
* most other IEEE 802 networks
* Fiber Distributed Data Interface (FDDI)
* Asynchronous Transfer Mode (ATM), switched virtual connections only, as part of an NSAP address
* Fibre Channel and Serial Attached SCSI (as part of a World Wide Name)
* The ITU-T G.hn standard, which provides a way to create a high-speed (up to 1 gigabit/s) local area network using existing home wiring (power lines, phone lines and coaxial cables). The G.hn Application Protocol Convergence (APC) layer accepts Ethernet frames that use the MAC-48 format and encapsulates them into G.hn Medium Access Control Service  Data Units (MSDUs).

### EUI-64###
* FireWire
* IPv6 (Modified EUI-64 as the least-significant 64 bits of a unicast network address or link-local address when stateless autoconfiguration is used)
* ZigBee / 802.15.4 / 6LoWPAN wireless personal-area networks

特殊MAC地址
----------------
###广播地址###
全F，在本VLAN内泛洪。
###IANA注册###
详见[ethernet-numbers](http://www.iana.org/assignments/ethernet-numbers/ethernet-numbers.xhtml)，此处仅截取常见地址。
####单播地址####
以`00-00-5e`开头：

* `00-01-00` to `00-01-FF`  VRRP (Virtual Router Redundancy Protocol)   [RFC5798]
* `00-02-00` to `00-02-FF`  VRRP IPv6 (Virtual Router Redundancy Protocol IPv6) [RFC5798]
* `90-01-00`    TRILL OAM   [RFC7455]

####组播地址####
以`01-00-5e`开头：

* `00-00-00` to `7F-FF-FF`  IPv4 Multicast  [RFC1112]
* `80-00-00` to `8F-FF-FF`  MPLS Multicast  [RFC5332]
* `90-00-00`    MPLS-TP p2p [RFC7213]
* `90-00-01`    Bidirectional Forwarding Detection (BFD) on Link Aggregation Group (LAG) Interfaces [RFC7130]
* `90-01-00`    TRILL OAM   [RFC7455]

映射组播IP为组播MAC地址转换
----------------------
如下图（来自微软）所示，IPv4组播MAC地址为IANA定义，前25位固定。而组播IP为D类IP地址，其前4位固定为1110b。将组播IP的低23位直接映射到MAC地址的低23位，即可与前面固定的25位组成一个48位的MAC地址。

由于组播IP还有5位没有映射到MAC地址中，因此组播MAC地址与组播IP不是一一映射，而是一对多的关系，一个组播MAC地址对应32位组播IP地址。

![Mapping IP Multicast to MAC-Layer Multicast](https://i-technet.sec.s-msft.com/dynimg/IC212799.gif)

> Written with [StackEdit](https://stackedit.io/).

