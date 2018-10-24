title: Redhat 系发行版本使用笔记
date: 2018-10-24 10:36:10
toc: true
tags: [Linux, redhat, centos]
categories: linux
keywords: [linux, redhat, centos, add user, networking, restart]
---

## 包管理

* 安装本地包，rpm -i xxx.rpm

## 网络相关

配置文件，`/etc/sysconfig/network-scripts`

```
[root@TENCENT64 /etc/sysconfig/network-scripts]# ls
eth1        ifcfg-eth4   ifdown-eth   ifdown-ppp     ifup-aliases  ifup-isdn   ifup-routes       network-functions
ifcfg-eth0  ifcfg-eth5   ifdown-ippp  ifdown-routes  ifup-bnep     ifup-plip   ifup-sit          network-functions-ipv6
ifcfg-eth1  ifcfg-lo     ifdown-ipv6  ifdown-sit     ifup-eth      ifup-plusb  ifup-tunnel       route-eth1
ifcfg-eth2  ifdown       ifdown-isdn  ifdown-tunnel  ifup-ippp     ifup-post   ifup-wireless     setdefaultgw-tlinux
ifcfg-eth3  ifdown-bnep  ifdown-post  ifup           ifup-ipv6     ifup-ppp    init.ipv6-global
```

重启网络服务，`# service network restart`

网卡配置，

```
[root@TENCENT64 /etc/sysconfig/network-scripts]# cat ifcfg-eth0
#IP Config for eth0:
DEVICE='eth0'
HWADDR=6c:92:bf:85:4c:ee
NM_CONTROLLED='yes'
ONBOOT='yes'
IPADDR='10.6.188.99'
NETMASK='255.255.255.0'
GATEWAY='10.6.188.1'
```

## Misc

* sudo 组不存在，添加 sudo 组，`groupadd sudo`，否则提示 `useradd: group 'sudo' does not exist`
* 添加用户，`useradd -G sudo,docker jeromesun -m -s /bin/bash`
* 添加用户到 sudoers file，在 root 用户下，visudo，添加一行 `jeromesun       ALL=(ALL)       ALL`。否则出现 `jeromesun is not in the sudoers file.  This incident will be reported.`。
