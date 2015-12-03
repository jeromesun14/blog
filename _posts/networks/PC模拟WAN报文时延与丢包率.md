title: PC模拟WAN报文时延与丢包率
date: 2015-10-12 21:10:18
toc: false
tags: [网络测试]
categories: networks
keywords: [computer networks test, 模拟WAN, 时延, 丢包]
description: Linux模拟WAN报文时延与丢包率的方法。
---

使用PC模拟WAN报文时延与丢包率的思路为：
* 报文进入PC，在PC中进行软件转发。
* 在PC的软件转发策略中，加入时延与丢包率。

测试方法：
* 多网卡，可简单控制某端口入、某端口出。一般可基于端口、流进行配置。
* 单网卡，PC机内部使用多虚拟网卡中转模拟WAN，在外部交换机进行PBR策略，对报文入口是PC的走另一条路由。

<!--more-->

工具：（这里只介绍两种）
* `netem` + `tc`，目前大多数Linux发行版天然支持，命令行使用上有学习成本。`netem`为内核上的功能，用户空间命令`tc`最终使用`netem`提供的服务。
* `WANem`，一个Linux版本，需要安装新系统。可通过网页进行配置。

测试须知：
* 由于PC的软件处理能力有限，测试时尽可能避免大流量软件转发，造成PC处理能力不足而引起的丢包、时延不准确。

`netem` + `tc`样例：
* 配置路由，`sudo ip route add xxx.xxx.xxx.xxx/mask -i ethx`
* 使能PC进行IP报文转发， `sudo sysctl net.ipv4.ip_forward=1`
* 查看端口收发包，`sudo tcpdump -i ethx` 
* 使用tc进行时延/丢包率配置，`sudo tc qdisc add dev eth0 root netem delay 100ms`

