title: p4app 仓库 —— 用于快速验证 P4 程序
date: 2017-05-10 16:41:00
toc: true
tags: [p4]
categories: p4
keywords: [p4, p4app, 快速验证, p4lang, github, repo, topo, stf, bmv2, build, compile, docker]
description: p4app 仓库介绍
---

## 简介
2017-02-23 p4lang 新建了一个 [p4app](https://github.com/p4lang/p4app) 仓库，用于快速、简便地构建、运行、调试和测试 P4 程序。摆脱 [p4factory](https://github.com/p4lang/p4factory) / [switch](https://github.com/p4lang/switch) 依赖安装步骤复杂、编译时间长的问题。降低入门门槛。与预期的类似，p4app 将 bmv2 等基础组件打成 docker image。

> p4app is a tool that can build, run, debug, and test P4 programs. The philosophy behind p4app is "easy things should be easy" - p4app is designed to make small, simple P4 programs easy to write and easy to share with others.

docker 镜像目前只有 908MB，已集成 tshark / scapy 等工具，不算大。

```
 sunyongfeng  ~  workshop  p4app  docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
p4lang/p4app          latest              07024040be0e        6 hours ago         908MB
```

## 使用步骤

只需要如下步骤就可以测试你的 P4 程序：

1. 安装 [docker](https://docs.docker.com/engine/installation/)
2. 拷贝 p4app 到系统 PATH 路径，例如 `sudo cp p4app /usr/local/bin`
3. p4app bug: [pull request 17](https://github.com/p4lang/p4app/pull/17)，docker 镜像名已由 `p4lang/p4app:stable` 改为 `p4lang/p4app:latest`。修改 `p4app`，将 `-P4APP_IMAGE=${P4APP_IMAGE:-p4lang/p4app:stable}` 改为 `-P4APP_IMAGE=${P4APP_IMAGE:-p4lang/p4app:latest}` 即可。
4. 运行 app，例如 simple_router，`p4app run examples/simple_router.p4app` 。运行后，如果系统没有 `p4lang/p4app:latest` image，则会自动下载。

## 主要构成

p4app 将 p4 代码、测试拓扑配置、测试用例打包在一个后续为 `.p4app` 的文件中。包含：

* `p4app.json`，用于配置程序名、P4 语言版本、测试拓扑配置、测试用例等
* `*.p4`，P4 源代码
* `*.stf`，simple testing framework，应属于测试用例的范畴，目前还没有看懂其语法
* `*.config`，bmv2 switch 的默认配置

## simple_router 样例运行 log

```
 sunyongfeng  ~  workshop  p4app  sudo cp p4app /usr/local/bin/
 sunyongfeng  ~  workshop  p4app  p4app run examples/simple_router.p4app/
Unable to find image 'p4lang/p4app:latest' locally
latest: Pulling from p4lang/p4app
c62795f78da9: Already exists 
d4fceeeb758e: Already exists 
5c9125a401ae: Already exists 
0062f774e994: Already exists 
6b33fd031fac: Already exists 
c0bc589d658e: Pull complete 
d304998c3f88: Pull complete 
9a10b2f1580d: Pull complete 
0f3b81cccf80: Pull complete 
d32e2c3a5556: Pull complete 
421d6ade42fb: Pull complete 
ed336ba6d94a: Pull complete 
15d9b8c3dfdb: Pull complete 
d7df888c0624: Pull complete 
09b889a34f73: Pull complete 
8e46ef6cc28e: Pull complete 
53625fe57a43: Pull complete 
a1cfe7751ecf: Pull complete 
098b357129cf: Pull complete 
ecb46f207e83: Pull complete 
25df5d0fdf2a: Pull complete 
a2592a836f76: Pull complete 
ba5f1dc63b79: Pull complete 
d8fe27124ef4: Pull complete 
b6c4e12d947a: Pull complete 
24342178e654: Pull complete 
21146199f8e0: Pull complete 
41e02212e6ab: Pull complete 
885ba0b6073a: Pull complete 
28805cf1e6e6: Pull complete 
d5f002ec605e: Pull complete 
07b6dd17ef27: Pull complete 
b91909b6c11f: Pull complete 
f0f07a64cc7e: Pull complete 
202436ed05da: Pull complete 
Digest: sha256:7d8a140c7160992f693b9d5fab08f7aeaacdb448050d17da81c4ae05dd5465d0
Status: Downloaded newer image for p4lang/p4app:latest
Entering build directory.
Extracting package.
Reading package manifest.
> p4c-bm2-ss --p4v 16 "simple_router.p4" -o "simple_router.p4.json"
> python2 "/scripts/mininet/single_switch_mininet.py" --log-file "/var/log/simple_router.p4.log" --cli-message "mininet_message.txt" --num-hosts 2 --switch-config "simple_router.config" --behavioral-exe "simple_switch" --json "simple_router.p4.json"
Adding host h1
Adding host h2
*** Error setting resource limits. Mininet's performance may be affected.
*** Creating network
*** Adding hosts:
h1 h2 
*** Adding switches:
s1 
*** Adding links:
(h1, s1) (h2, s1) 
*** Configuring hosts
h1 h2 
*** Starting controller

*** Starting 1 switches
s1 Starting P4 switch s1.
simple_switch -i 1@s1-eth1 -i 2@s1-eth2 --thrift-port 9090 --nanolog ipc:///tmp/bm-0-log.ipc --device-id 0 simple_router.p4.json --debugger --log-console
P4 switch s1 has been started.

**********
Network configuration for: h1
Default interface: h1-eth0	10.0.0.10	00:04:00:00:00:00
Default route to switch: 10.0.0.1 (00:aa:bb:00:00:00)
**********
**********
Network configuration for: h2
Default interface: h2-eth0	10.0.1.10	00:04:00:00:00:01
Default route to switch: 10.0.1.1 (00:aa:bb:00:00:01)
**********

Reading switch configuration script: simple_router.config
Configuring switch...
Obtaining JSON from switch...
Done
Control utility for runtime P4 table manipulation
RuntimeCmd: Setting default action of send_frame
action:              _drop
runtime data:        
RuntimeCmd: Setting default action of forward
action:              _drop
runtime data:        
RuntimeCmd: Setting default action of ipv4_lpm
action:              _drop
runtime data:        
RuntimeCmd: Adding entry to exact match table send_frame
match key:           EXACT-00:01
action:              rewrite_mac
runtime data:        00:aa:bb:00:00:00
Entry has been added with handle 0
RuntimeCmd: Adding entry to exact match table send_frame
match key:           EXACT-00:02
action:              rewrite_mac
runtime data:        00:aa:bb:00:00:01
Entry has been added with handle 1
RuntimeCmd: Adding entry to exact match table forward
match key:           EXACT-0a:00:00:0a
action:              set_dmac
runtime data:        00:04:00:00:00:00
Entry has been added with handle 0
RuntimeCmd: Adding entry to exact match table forward
match key:           EXACT-0a:00:01:0a
action:              set_dmac
runtime data:        00:04:00:00:00:01
Entry has been added with handle 1
RuntimeCmd: Adding entry to lpm match table ipv4_lpm
match key:           LPM-0a:00:00:0a/32
action:              set_nhop
runtime data:        0a:00:00:0a	00:01
Entry has been added with handle 0
RuntimeCmd: Adding entry to lpm match table ipv4_lpm
match key:           LPM-0a:00:01:0a/32
action:              set_nhop
runtime data:        0a:00:01:0a	00:02
Entry has been added with handle 1
RuntimeCmd: 
Configuration complete.

Ready !

======================================================================
Welcome to the BMV2 Mininet CLI!
======================================================================
Your P4 program is installed into the BMV2 software switch
and your initial configuration is loaded. You can interact
with the network using the mininet CLI below.

To inspect or change the switch configuration, connect to
its CLI from your host operating system using this command:
  docker exec -t -i 66d97e58a61f simple_switch_CLI

To view the switch log, run this command from your host OS:
  docker exec -t -i 66d97e58a61f tail -f /var/log/simple_router.p4.log

To run the switch debugger, run this command from your host OS:
  docker exec -t -i 66d97e58a61f bm_p4dbg


*** Starting CLI:
mininet> h1 ping h2
PING 10.0.1.10 (10.0.1.10) 56(84) bytes of data.
64 bytes from 10.0.1.10: icmp_seq=1 ttl=63 time=1.42 ms
64 bytes from 10.0.1.10: icmp_seq=2 ttl=63 time=1.65 ms
^C
--- 10.0.1.10 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 1.421/1.540/1.659/0.119 ms
mininet> 
mininet> 
mininet> exit
*** Stopping 0 controllers

*** Stopping 2 links
..
*** Stopping 1 switches
s1 
*** Stopping 2 hosts
h1 h2 
*** Done
 sunyongfeng  ~  workshop  p4app  
```

