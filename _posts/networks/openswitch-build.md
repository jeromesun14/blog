title: openswitch 编译记录
date: 2016-03-18 12:30:30
toc: true
tags: [NOS, openswitch]
categories: networks
keywords: [openswitch, 编译]
description: 介绍开源网络系统 openswitch 编译过程。
---

> From [link](http://therandomsecurityguy.com/openvswitch-cheat-sheet/)。OVS is feature rich with different configuration commands, but the majority of your configuration and troubleshooting can be accomplished with the following 4 commands:
* ovs-vsctl : Used for configuring the ovs-vswitchd configuration database (known as ovs-db)
* ovs-ofctl : A command line tool for monitoring and administering OpenFlow switches
* ovs-dpctl : Used to administer Open vSwitch datapaths
* ovs−appctl : Used for querying and controlling Open vSwitch daemons

## 编译步骤

* 安装依赖软件包，在 Ubuntu 14.04 上的依赖软件包有：
  + screen
  + chrpath
  + device_tree_compiler
  + gawk
  + makeinfo，软件包名字是 `texinfo`

```
sudo apt-get install screen chrpath device_tree_compiler gawk texinfo
```

* 下载 `ops-build`

```
git clone https://git.openswitch.net/openswitch/ops-build [<directory>]
cd <directory>
```

* 配置 openswitch 产品

```
make configure genericx86-64
```

* 编译产品
注意需要编译较长时间。

```
make
```

* 部署产品
XXX: todo
