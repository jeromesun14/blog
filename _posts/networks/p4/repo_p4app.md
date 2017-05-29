title: p4app 仓库 —— 用于快速验证 P4 程序
date: 2017-05-10 16:41:00
toc: true
tags: [p4]
categories: p4
keywords: [p4, p4app, 快速验证, p4lang, github, repo, topo, stf, bmv2, build, compile, docker, mininet]
description: p4app 仓库使用说明。
---

## 简介
p4lang 于 2017-02-23 创建 [p4app](https://github.com/p4lang/p4app) 仓库，用于快速、简便地构建、运行、调试和测试 P4 程序。

 [p4factory](https://github.com/p4lang/p4factory) 或 [switch](https://github.com/p4lang/switch) 等 P4 样例依赖安装步骤复杂、编译时间超长。p4app 可很好地解决此问题，降低 P4 入门门槛。与预期的相符，p4app 将 bmv2 等基础组件整合成一个 docker image。

> p4app 是一种构建、运行、调试和测试 P4 程序的工具。其背后的哲学是“简单的东西应尽可能简单”。p4app 旨在使小而简单的 P4 程序易于编写、易于与他人分享。

安装
-------

* 下载 p4app

```
git clone https://github.com/p4lang/p4app
```

* Bug fix.

p4app bug: [pull request 17](https://github.com/p4lang/p4app/pull/17)，docker 镜像名已由 `p4lang/p4app:stable` 改为 `p4lang/p4app:latest`。修改 `p4app`，将：

```
-P4APP_IMAGE=${P4APP_IMAGE:-p4lang/p4app:stable}
``` 

修改为：

```
-P4APP_IMAGE=${P4APP_IMAGE:-p4lang/p4app:latest}
```

* 如果没有则需先安装 [docker](https://docs.docker.com/engine/installation/) 。

* 如果需要，可以把 `p4app` 脚本放到 PATH 路径，例如：

```
sudo cp p4app /usr/local/bin
```

程序安装到此结束！

使用
------

p4app 运行以 `.p4app` 为后缀的目录，称 p4app 包。p4app 仓库中包含多个样例，例如 `simple_router.p4app`，以下是运行它的方法：

```
p4app run examples/simple_router.p4app
```

该命令最终进入 mininet 命令行，这此之前，p4app 进行如下处理：

1. 【首次运行 p4app 命令时】自动下载 docker 镜像 `p4lang/p4app:latest`，该镜像包含 P4 编译器、抓包工具 `tshark`、发包工具 `scapy`、net-tools 和 nmap 套件等工具
2. 编译 `simper_router.p4`
3. 设置并启动一个容器做为软件交换机
4. 设置并启动 mininet 模拟实验网络

## p4app 命令参数

* run，运行 p4app，可带 target 名。
* pack，压缩 p4app 包为单独文件，便于分享。使用 gzip 压缩
* unpack，解压上述压缩包
* update，更新 p4app 本地缓存的 P4 编译器和相关工具到最新版本。p4app 在本地缓存P4编译器和工具，因此不必每次都重新下载。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ ./p4app -h
Usage:
  p4app run <program.p4app>
      Run a p4app.
  p4app run <program.p4app> <target>
      Run a p4app, specifying a target.
  p4app pack <program.p4app>
      Compress a p4app directory into a single file, in-place.
  p4app unpack <program.p4app>
      Expand a p4app file into a directory, in-place.
  p4app update
      Update the toolchain to the newest version.
```

### target 与 backend
p4app 包最终将怎么运行，由配置文件 manifest file `p4app.json` 中的 `targets` 参数指定。目前支持有多种运行方式，例如仅编译成 bmv2 可用目标、进行 stf 测试、mininet 单交换机测试、mininet 多交换机测试等。本文将这些运行方式称为 backend，一个 p4app 包可以有多个 target，每个 target 必须指定其 backend。

通过在 `p4app run` 时指定 target 运行非默认 target，例如运行 simple_counter.p4app 的 debug target：

```
p4app run examples/simple_couter.p4app debug
```

目前支持以下几种 backend，具体见后文详述。

* mininet
* multiswitch
* stf
* compile-bmv2


创建一个 p4app 包
------------------------
p4app 包的目录结构看起来像这样：

```
  my_program.p4app
    |
    |- p4app.json
    |
    |- my_program.p4
    |
    |- ...other files...
```

`p4app.json`  是这个包的 manifest，说明如何构建和运行 p4 程序，功能有点像 Markfile 文件。样例：

```
{
  "program": "my_program.p4",
  "language": "p4-14",
  "targets": {
    "mininet": {
      "num-hosts": 2,
      "switch-config": "my_program.config"
    }
  }
}
```

样例 manifest 告诉 p4app 应该运行  `my_program.p4`，该 p4 程序使用 `p4-14` 编写（同样也可以用 `p4-16`，P4-16 草案版本）。该样例定义一个 target，该 target 的 backend 为 `mininet`，同时提供一些 Mininet 配置选项：测试网络有两个 host，一个模拟交换机，该交换机启机默认加载 `my_program.config` 中的配置。如果你在 `p4app.json` 中引用像 `my_program.config` 时，则需要将 `my_program.config` 文件包在 p4app 包中，p4app 将确保相应工具可找到它。

如果有多个 target，且使用者没有通过名字指定默认 target，则 p4app 随机运行其中一个。可使用 `default-target` 选项，设置默认 backend。例如：

```
{
  "program": "my_program.p4",
  "language": "p4-14",
  "default-target": "debug",
  "targets": {
    "debug": { "use": "mininet", "num-hosts": 2 },
    "test1": { "use": "stf", "test": "test1.stf" },
    "test2": { "use": "stf", "test": "test2.stf" },
  }
}
```

这里定义一个名为 “debug” 的 Mininet backend，以及两个 STF backend，分别名为“test1”和“test2”。`"user":"mininet"` 用于指明每个 target 使用哪个 backend，如果不使用 user 字段，则 target 名同时被认定为 backend 名。这也是为什么，在之前的样例中，不需要指明`"use": "mininet"` ，因为 target 名就已经是 mininet，如果直接被用成 backend 名，p4app 也可以识别。

Backend
------------

### mininet

该后端编译 p4 程序，并加载到 BMv2 [simple_switch](https://github.com/p4lang/behavioral-model/blob/master/docs/simple_switch.md)，然后创建 Mininet 实验环境。

支持以下配置值（皆为可选）：

```
"mininet": {
  "num-hosts": 2,
  "switch-config": "file.config"
}
```

Mininet 将通过星形拓扑创建网络，并创建 `num-hosts` 个数的 host，通过不同端口连接到模拟交换机。

模拟交换机的启机默认配置可通过 `switch-config` 配置。配置的文件是一系列 BMV2 [simple_switch_CLI](https://github.com/p4lang/behavioral-model#using-the-cli-to-populate-tables) 命令。

在启机过程中，有信息提示网络配置信息以及如何使用 logging 和 debugging 工具。

BMV2 debugger 很方便，可在[这里](https://github.com/p4lang/behavioral-model/blob/master/docs/p4dbg_user_guide.md) 阅读如何使用。

该 backend 还支持`compile-bmv2` 配置编译选项，详见下文相应章节。

### multiswitch

和 `mininet` 一样，这个 target 包含 P4 程序，并支行在 Mininet 环境。但是这个 backend 支持配置多个交换机、自定义拓扑并在 host 上执行自定义命令。其中这些交换机默认自动配置 l2/l3 规则，用于所有 host 之间的互通，即假设 P4 程序有 `ipv4_lpm`、`send_frame` 和 `forward` 表，详见 simple_router.p4。

配置样例：

```
"multiswitch": {
  "links": [
    ["h1", "s1"],
    ["s1", "s2"],
    ["s2", "h2", 50]
  ],
  "hosts": {
    "h1": {
      "cmd": "python echo_server.py $port",
      "startup_sleep": 0.2,
      "wait": false
    },
    "h2": {
      "cmd": "python echo_client.py h1 $port $echo_msg",
      "wait": true
    }
  },
  "parameters": {
    "port": 8000,
    "echo_msg": "foobar"
  }
}
```

该配置创建以下拓扑：

```
h1 <---> s1 <---> s2 <---> h2
```

其中 `s2-h2` 链路人工配置 50ms 的延迟。host 配置选项：

 - `cmd` - 在 host 上运行的命令
 - `wait` - 等待命令执行结束。如果配置成 false，表示在后台运行此命令。
 - `startup_sleep` - 启动命令后应等待的时间（以秒为单位）。
 - `latency` - 主机与交换机之间的延迟。配置值是数字（解释为秒）或具有时间单位的字符串（例如`50ms`或`1s`）。该配置将覆盖“links”对象中设置的延迟。

通过将主机名（例如“h1”）替换为相应的IP地址来格式化该命令。 目标中指定的参数将作为环境变量（即`$`后跟变量名称）可用于命令，详见 multiswitch 样例。

#### 限制
目前每个 host 最多只能连一个 switch。

#### 指定每个交换机表项
路由表（`ipv4_lpm`，`send_frame` 和 `forward`）在本 target 中自动下发。使用者可根据自己的需要下发表项到每个交换机中，形式可以为包含一个命令文件或命令数组。这些自定义的表项优先级比路由表的自动生成的高。例如：

```
"multiswitch": {
  "links": [ ... ],
  "hosts": { ... },
  "switches": {
    "s1": {
      "entries": "s1_commands.txt"
    },
    "s2": {
      "entries": [
        "table_add ipv4_lpm set_nhop 10.0.1.10/32 => 10.0.1.10 1",
        "table_add ipv4_lpm set_nhop 10.0.2.10/32 => 10.0.2.10 2"
      ]
    }
  }
}
```

如果上述 `s2` 表项与自动生成的表项一样（例如自动生成的表项 `set_nhop 10.0.1.10 / 32`），这些自定义表项将具有更高的优先权，在下发表项时将警告表项重复。

#### 自定义拓扑类

可通过 `topo_module` 选项指定自己的 mininet 拓扑类。例如：

```
"multiswitch": {
  ...
  "topo_module": "mytopo"
  ...
}
```

这将 import `mytopo` 模块 `mytopo.py`，该文件应与 manifest 文件（`p4app.json`）放在同一目录下，同时应实现 `CustomAppTopo` 类。可 extend 默认的 topo 类 [apptopo.AppTopo](https://github.com/p4lang/p4app/blob/master/docker/scripts/mininet/apptopo.py)。例如：

```
# mytopo.py
from apptopo import AppTopo

class CustomAppTopo(AppTopo):
    def __init__(self, *args, **kwargs):
        AppTopo.__init__(self, *args, **kwargs)

        print self.links()
```

详见样例 [customtopo.p4app](https://github.com/p4lang/p4app/blob/master/examples/customtopo.p4app/mytopo.py)。

#### 自定义控制器
类似 `topo_module` 选项，可通过 `controller_module` 选项自定义控制器。该模块应实现 `CustomAppController` 类。默认的控制器类为 [appcontroller.AppController](https://github.com/p4lang/p4app/blob/master/docker/scripts/mininet/appcontroller.py)，可直接对该类进行扩展。
例如，样例 [customtopo.p4app](https://github.com/p4lang/p4app/blob/master/examples/customtopo.p4app/mycontroller.py)。

#### Logging

当本 target 运行时，host 上的临时目录 `/tmp/p4app_log` 将加载到 guest 的 `/tmp/p4app_log`。运行 p4app 后， host 上保存有所有的 log。 host 命令的标准输出 stdout 将保存在该目录，如果需要保存某个命令的 log，将可以将其输出到该目录。

设置 `"bmv2_log":true`，保存 P4 交换机的调试 log。
设置 `"pcap_dump":true`，抓取所有交换机的报文，交保存为 PCAP 格式。

以上文件将被保存到 `/tmp/p4app_log`。详见样例 [broadcast.p4app](https://github.com/p4lang/p4app/blob/master/examples/broadcast.p4app/p4app.json) 

#### Cleanup commands
运行 target 后（在 mininet stop 之前），如果需要在 docker 容器内执行命令，可使用 `after` 选项，其后必须包含 `cmd`，可以是一个或多个命令，例如：

```
"multiswitch": {
  "links": [ ... ],
  "hosts": { ... },
  "after": {
    "cmd": [
      "echo register_read my_register 1 | simple_switch_CLI --json p4src/my_router.p4.json",
      "echo register_read my_register 2 | simple_switch_CLI --json p4src/my_router.p4.json"
    ]
  }
}
```

### custom
此为第三种 backend，允许使用者指定 python `program`，该 `program` 使用 Mininet python API 指定网络拓扑和配置。例如：

```
{
  "program": "source_routing.p4",
  "language": "p4-14",
  "targets": {
      "custom": {
	       "program": "topo.py"
      }
  }
}
```

该 target 在启动 Mninet 时运行 `topy.py` 脚本。将使用以下参数调用 `program`

| 参数             | 说明 |
| --------         | ----------- |
| --behavioral-exe | switch 可执行文件 |
| --json           | P4 程序编译结果 |
| --cli            | switch_CLI 命令|

样例：

```
PYTHONPATH=$PYTHONPATH:/scripts/mininet/ python2 topo.py \
                        --behavioral-exe simple_switch \
                        --json SOME_FILE \
                        --cli simple_switch_CLI
```

同时可以指定其他参数传递给自定义拓扑程序，方法是将它们包含在 `program` 定义中，如下所示：

```
{
  "program": "source_routing.p4",
  "language": "p4-14",
  "targets": {
      "custom": {
	       "program": "topo.py --num-hosts 2 --switch-config simple_router.config"
      }
  }
}

```

`program` 可通过 `HOSTNAME` 变量获取 docker 容器 ID。

```python
import os
container = os.environ['HOSTNAME']
print 'Run the switch CLI as follows:'
print '  docker exec -t -i %s %s' % (container, args.cli)
```

### stf

stf 后端编译给定的 p4 程序，并运行 STF 测试用例。

stf，simple testing framework，一种模拟网络测试框架。

> "simple testing framework", which can help you test small P4 programs and ensure they behave the way you expect.

`simple_counter.p4app` 样例使用 stf 测试框架，运行该样例：

```
p4app run examples/simple_counter.p4app
```

运行后，p4app 启动 docker 容器，并在容器中：

* 自动编译 `simple_counter.p4`
* 运行 simple_switch 交换机
* stf 下发默认表项
* stf 发送定义在 `simple_counter.stf` 中的报文，并验证 simple_switch 是否转发出预期的报文

和 mininet 不一样的是， stf 最终直接退出。

#### simple_counter.p4app stf 配置

* `p4app.json` 配置

```json
{
  "program": "simple_counter.p4",
  "language": "p4-14",
  "targets": {
    "stf": {
      "test": "simple_counter.stf"
    },
    "debug": {
      "use": "mininet",
      "num-hosts": 2,
      "switch-config": "simple_counter.config"
    }
  }
}
```

* stf 文件 simple_counter.stf 配置

```
add test1 data.f1:0x01010101 c1_2(val1:0x01, val2:0x02)
add test1 data.f1:0x02020202 c1_2(val1:0x10, val2:0x20)

add test2 data.f2:0x03030303 c3_4(val3:0x03, val4:0x04, port:1)
add test2 data.f2:0x04040404 c3_4(val3:0x30, val4:0x40, port:2)

expect 1 01010101 03030303 01 02 03 04
packet 0 01010101 03030303 55 66 77 88
expect 2 01010101 04040404 01 02 30 40
packet 0 01010101 04040404 55 66 77 88
expect 1 02020202 03030303 10 20 03 04
packet 0 02020202 03030303 99 88 77 66
expect 2 02020202 04040404 10 20 30 40
packet 0 02020202 04040404 14 25 36 47
```

必须使用 STF 格式编写 stf target 指定的配置文件，目前还没有该格式的说明文档。（如果您想反向工程并提供一些文档，请提交PR！）请参阅 p4app 相关样例，参考其中的基本用法。

此后端还支持`compile-bmv2`目标的配置值。

### compile-bmv2

这是一个简单的后端，将 p4 程序编译成 BMV2 目标。

支持以下可选配置值：

```
"compile-bmv2": {
  "compiler-flags": ["-v", "-E"],
  "run-before-compile": ["date"],
  "run-after-compile": ["date"]
}
```

高级功能
--------

### 指定 docker image

如果你想使用自己的 P4 工具链或 p4app，可通过 `P4APP_IMAGE` 环境变量配置 Docker image，替代标准 p4lang image。例如：

```
P4APP_IMAGE=me/my_p4app_image:latest p4app run examples/simple_router.p4app
```

### 指定 manifest 文件

默认情况下，p4app 使用 app 目录下一个名为 `p4app.json` 的 manifest 文件。如果你的 manifest 文件名称不是 `p4app.json`，则可通过 `--manifest` 选项指定 manifest 文件。例如：

```
p4app run myapp.p4app --manifest testing.p4app
```

### 指定 log 目录
默认情况下，p4app 挂载 host 目录 `/tmp/p4app_logs` 到 docker 容器 guest 目录 `/tmp/p4app_logs`。bmv2 以及其他程序的输出将保存在此目录。可通过 `$P4APP_LOGDIR` 环境变量指定其他目录为 log 目录，例如：

```
P4APP_LOGDIR=./out p4app run myapp.p4app
```


运行 log 汇总
----

### 运行 simple_router.p4app

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ sudo cp p4app /usr/local/bin/
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ p4app run examples/simple_router.p4app/
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
```

### 运行 simple_counter.p4app

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ p4app run examples/simple_counter.p4app/
Entering build directory.
Extracting package.
Reading package manifest.
> p4c-bm2-ss --p4v 14 "simple_counter.p4" -o "simple_counter.p4.json"
warning: cnt: Direct counter not used; ignoring
> python2 "/scripts/stf/bmv2stf.py" -v /tmp/simple_counter.p4.json /tmp/simple_counter.stf
WARNING: No route found for IPv6 destination :: (no default route?)
Running model
Running simple_switch --log-file switch.log --log-flush --use-files 0 --thrift-port 9232 --device-id 142 -i 0@pcap0 -i 1@pcap1 -i 2@pcap2 /tmp/simple_counter.p4.json
Calling target program-options parser
Adding interface pcap0 as port 0 (files)
Adding interface pcap1 as port 1 (files)
Adding interface pcap2 as port 2 (files)
Running simple_switch_CLI --thrift-port 9232
Thrift server was started
STF Command: 
STF Command: add test1 data.f1:0x01010101 c1_2(val1:0x01, val2:0x02)
table_add test1 c1_2 0x01010101 => 0x01 0x02
STF Command: add test1 data.f1:0x02020202 c1_2(val1:0x10, val2:0x20)
table_add test1 c1_2 0x02020202 => 0x10 0x20
STF Command: 
STF Command: add test2 data.f2:0x03030303 c3_4(val3:0x03, val4:0x04, port:1)
table_add test2 c3_4 0x03030303 => 0x03 0x04 1
STF Command: add test2 data.f2:0x04040404 c3_4(val3:0x30, val4:0x40, port:2)
table_add test2 c3_4 0x04040404 => 0x30 0x40 2
STF Command: 
STF Command: expect 1 01010101 03030303 01 02 03 04
STF Command: packet 0 01010101 03030303 55 66 77 88
Obtaining JSON from switch...
Done
Control utility for runtime P4 table manipulation
RuntimeCmd: Adding entry to exact match table test1
match key:           EXACT-01:01:01:01
action:              c1_2
runtime data:        01 02
Entry has been added with handle 0
RuntimeCmd: Adding entry to exact match table test1
match key:           EXACT-02:02:02:02
action:              c1_2
runtime data:        10 20
Entry has been added with handle 1
RuntimeCmd: Adding entry to exact match table test2
match key:           EXACT-03:03:03:03
action:              c3_4
runtime data:        03 04      00:01
Entry has been added with handle 0
RuntimeCmd: Adding entry to exact match table test2
match key:           EXACT-04:04:04:04
action:              c3_4
runtime data:        30 40      00:02
Entry has been added with handle 1
STF Command: expect 2 01010101 04040404 01 02 30 40
STF Command: packet 0 01010101 04040404 55 66 77 88
STF Command: expect 1 02020202 03030303 10 20 03 04
STF Command: packet 0 02020202 03030303 99 88 77 66
STF Command: expect 2 02020202 04040404 10 20 30 40
STF Command: packet 0 02020202 04040404 14 25 36 47
RuntimeCmd: 
simple_switch exit code -15
Execution completed
Comparing outputs
WARNING: PcapReader: unknown LL type [0]/[0x0]. Using Raw packets
WARNING: PcapReader: unknown LL type [0]/[0x0]. Using Raw packets
SUCCESS
```

### pack & unpack 样例 log

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ ls multiswitch.p4app/
echo_client.py  echo_server.py  header.p4  p4app.json  parser.p4  simple_router.p4
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ p4app pack multiswitch.p4app/  
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ ls
broadcast.p4app     customtopo.p4app   multiswitch.p4app     simple_counter.p4app  source_routing.p4app
compile_only.p4app  multi_iface.p4app  paxos_acceptor.p4app  simple_router.p4app
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ file multiswitch.p4app 
multiswitch.p4app: gzip compressed data, last modified: Mon May 29 08:25:21 2017, from Unix
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ p4app unpack multiswitch.p4app 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ ls multiswitch.p4app/
echo_client.py  echo_server.py  header.p4  p4app.json  parser.p4  simple_router.p4
```

### compile-bmv2 backend 样例

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ p4app run examples/compile_only.p4app/
Entering build directory.
Extracting package.
Reading package manifest.
> p4c-bm2-ss --p4v 14 -v --p4runtime-file out.bin --p4runtime-as-json "compile_only.p4" -o "compile_only.p4.json"
error: Unknown option --p4runtime-as-json
p4c-bm2-ss: Compile a P4 program
--help                                  Print this help message
--version                               Print compiler version
-I path                                 Specify include path (passed to preprocessor)
-D arg=value                            Define macro (passed to preprocessor)
-U arg                                  Undefine macro (passed to preprocessor)
-E                                      Preprocess only, do not compile (prints program on stdout)
--nocpp                                 Skip preprocess, assume input file is already preprocessed.
--p4v {14|16}                           Specify language version to compile
--target target                         Compile for the specified target
--pp file                               Pretty-print the program in the specified file.
--toJSON file                           Dump IR to JSON in the specified file.
--testJson                              Dump and undump the IR
--p4runtime-file file                   Write a P4Runtime control plane API description to the specified file.
--p4runtime-format {binary,json,text}   Choose output format for the P4Runtime API description (default is binary).
-o outfile                              Write output to outfile
--Werror                                Treat all warnings as errors
-T loglevel                             [Compiler debugging] Adjust logging level per file (see below)
-v                                      [Compiler debugging] Increase verbosity level (can be repeated)
--top4 pass1[,pass2]                    [Compiler debugging] Dump the P4 representation after
                                        passes whose name contains one of `passX' substrings.
                                        When '-v' is used this will include the compiler IR.
--dump folder                           [Compiler debugging] Folder where P4 programs are dumped
loglevel format is:
  sourceFile:level,...,sourceFile:level
where 'sourceFile' is a compiler source file and
'level' is the verbosity level for LOG messages in that file
> cat out.bin
cat: out.bin: No such file or directory
Compile failed.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ vi examples/compile_only.p4app/p4app.json
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ p4app run examples/compile_only.p4app/
Entering build directory.
Extracting package.
Reading package manifest.
> p4c-bm2-ss --p4v 14 -v --p4runtime-file out.bin --p4runtime-format json "compile_only.p4" -o "compile_only.p4.json"
Invoking preprocessor 
cpp -C -undef -nostdinc  -D__TARGET_BMV2__ -I/usr/local/share/p4c/p4_14include compile_only.p4
Invoking preprocessor 
cpp -C -undef -nostdinc  -I/usr/local/share/p4c/p4include /usr/local/share/p4c/p4include/v1model.p4
Parsing P4-16 program /usr/local/share/p4c/p4include/v1model.p4
Parsing P4-14 program compile_only.p4
Converting to P4-16
Converter_0_DoConstantFolding
Converter_1_CheckHeaderTypes
Converter_2_TypeCheck
Converter_3_DiscoverStructure
Converter_4_ComputeCallGraph
Converter_5_Rewriter
Converter_6_FixExtracts
FrontEnd_0_PrettyPrint
FrontEnd_1_ValidateParsedProgram
FrontEnd_2_CreateBuiltins
FrontEnd_3_ResolveReferences
FrontEnd_4_ConstantFolding
FrontEnd_5_InstantiateDirectCalls
FrontEnd_6_ResolveReferences
FrontEnd_7_TypeInference
FrontEnd_8_BindTypeVariables
FrontEnd_9_ClearTypeMap
FrontEnd_10_TableKeyNames
FrontEnd_11_ConstantFolding
FrontEnd_12_StrengthReduction
FrontEnd_13_UselessCasts
FrontEnd_14_SimplifyControlFlow
FrontEnd_15_FrontEndDump
FrontEnd_16_RemoveAllUnusedDeclarations
FrontEnd_17_SimplifyParsers
FrontEnd_18_ResetHeaders
FrontEnd_19_UniqueNames
FrontEnd_20_MoveDeclarations
FrontEnd_21_MoveInitializers
FrontEnd_22_SideEffectOrdering
FrontEnd_23_SetHeaders
FrontEnd_24_SimplifyControlFlow
FrontEnd_25_MoveDeclarations
FrontEnd_26_SimplifyDefUse
FrontEnd_27_UniqueParameters
FrontEnd_28_SimplifyControlFlow
FrontEnd_29_SpecializeAll
FrontEnd_30_RemoveParserControlFlow
FrontEnd_31_FrontEndLast
MidEnd_0_ConvertEnums
MidEnd_1_VisitFunctor
MidEnd_2_RemoveReturns
MidEnd_3_MoveConstructors
MidEnd_4_RemoveAllUnusedDeclarations
MidEnd_5_ClearTypeMap
MidEnd_6_Evaluator
MidEnd_7_VisitFunctor
MidEnd_8_Inline
MidEnd_9_InlineActions
MidEnd_10_LocalizeAllActions
MidEnd_11_UniqueNames
MidEnd_12_UniqueParameters
MidEnd_13_SimplifyControlFlow
MidEnd_14_RemoveActionParameters
MidEnd_15_SimplifyKey
MidEnd_16_ConstantFolding
MidEnd_17_StrengthReduction
MidEnd_18_SimplifySelectCases
MidEnd_19_ExpandLookahead
MidEnd_20_SimplifyParsers
MidEnd_21_StrengthReduction
MidEnd_22_EliminateTuples
MidEnd_23_CopyStructures
MidEnd_24_NestedStructs
MidEnd_25_SimplifySelectList
MidEnd_26_RemoveSelectBooleans
MidEnd_27_Predication
MidEnd_28_MoveDeclarations
MidEnd_29_ConstantFolding
MidEnd_30_LocalCopyPropagation
MidEnd_31_ConstantFolding
MidEnd_32_MoveDeclarations
MidEnd_33_ValidateTableProperties
MidEnd_34_SimplifyControlFlow
MidEnd_35_CompileTimeOperations
MidEnd_36_TableHit
MidEnd_37_SynthesizeActions
MidEnd_38_MoveActionsToTables
MidEnd_39_TypeChecking
MidEnd_40_SimplifyControlFlow
MidEnd_41_RemoveLeftSlices
MidEnd_42_TypeChecking
MidEnd_43_LowerExpressions
MidEnd_44_ConstantFolding
MidEnd_45_TypeChecking
MidEnd_46_RemoveComplexExpressions
MidEnd_47_FixupChecksum
MidEnd_48_SimplifyControlFlow
MidEnd_49_RemoveAllUnusedDeclarations
MidEnd_50_Evaluator
MidEnd_51_VisitFunctor
MidEnd_52_MidEndLast
> cat out.bin
{
 "tables": [
  {
   "preamble": {
    "id": 33588198,
    "name": "t1",
    "alias": "t1"
   },
   "actionRefs": [
    {
     "id": 16792110
    },
    {
     "id": 16800567,
     "annotations": [
      "@default_only()"
     ]
    }
   ],
   "size": "1024"
  }
 ],
 "actions": [
  {
   "preamble": {
    "id": 16800567,
    "name": "NoAction",
    "alias": "NoAction"
   }
  },
  {
   "preamble": {
    "id": 16792110,
    "name": "a1",
    "alias": "a1"
   },
   "params": [
    {
     "id": 1,
     "name": "port",
     "bitwidth": 9
    }
   ]
  }
 ]
}
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ 
```

### 运行 log 样例
运行样例 bcast_router.p4app，虽然运行后有提示错误，但是不影响输出 log。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ p4app run broadcast.p4app/
Entering build directory.
Extracting package.
Reading package manifest.
> p4c-bm2-ss --p4v 16 "bcast_router.p4" -o "bcast_router.p4.json"
> python2 "/scripts/mininet/multi_switch_mininet.py" --log-dir "/tmp/p4app_logs" --manifest "./p4app.json" --target "multiswitch" --auto-control-plane --behavioral-exe "simple_switch" --json "bcast_router.p4.json"
*** Error setting resource limits. Mininet's performance may be affected.
*** Creating network
*** Adding hosts:
h1 h2 h3 
*** Adding switches:
s1 
*** Adding links:
(0ms delay) (0ms delay) (h1, s1) (0ms delay) (0ms delay) (h2, s1) (0ms delay) (0ms delay) (h3, s1) 
*** Configuring hosts
h1 h2 h3 
*** Starting controller

*** Starting 1 switches
s1 Starting P4 switch s1.
simple_switch -i 1@s1-eth1 -i 2@s1-eth2 -i 3@s1-eth3 --pcap --thrift-port 9090 --nanolog ipc:///tmp/bm-0-log.ipc --device-id 0 bcast_router.p4.json --log-console
P4 switch s1 has been started.

-----------------------
table_add ipv4_lpm broadcast 255.255.255.255/32 =>
table_add forward set_dmac 255.255.255.255 => ff:ff:ff:ff:ff:ff
mc_mgrp_create 1
mc_node_create 0 1 2 3
mc_node_associate 1 0
table_add ipv4_lpm broadcast 255.255.255.255/32 =>
table_add forward set_dmac 255.255.255.255 => ff:ff:ff:ff:ff:ff
mc_mgrp_create 1
mc_node_create 0 1 2 3
mc_node_associate 1 0
table_set_default send_frame _drop
table_set_default forward _drop
table_set_default ipv4_lpm _drop
table_add send_frame rewrite_mac 2 => 00:aa:00:01:00:02
table_add forward set_dmac 10.0.2.10 => 00:04:00:00:00:02
table_add ipv4_lpm set_nhop 10.0.2.10/32 => 10.0.2.10 2
table_add send_frame rewrite_mac 3 => 00:aa:00:01:00:03
table_add forward set_dmac 10.0.3.10 => 00:04:00:00:00:03
table_add ipv4_lpm set_nhop 10.0.3.10/32 => 10.0.3.10 3
table_add send_frame rewrite_mac 1 => 00:aa:00:01:00:01
table_add forward set_dmac 10.0.1.10 => 00:04:00:00:00:01
table_add ipv4_lpm set_nhop 10.0.1.10/32 => 10.0.1.10 1
Obtaining JSON from switch...
Done
Control utility for runtime P4 table manipulation
RuntimeCmd: Adding entry to lpm match table ipv4_lpm
match key:           LPM-ff:ff:ff:ff/32
action:              broadcast
runtime data:        
Entry has been added with handle 0
RuntimeCmd: Adding entry to exact match table forward
match key:           EXACT-ff:ff:ff:ff
action:              set_dmac
runtime data:        ff:ff:ff:ff:ff:ff
Entry has been added with handle 0
RuntimeCmd: Creating multicast group 1
RuntimeCmd: Creating node with rid 0 , port map 1110 and lag map 
node was created with handle 0
RuntimeCmd: Associating node 0 to multicast group 1
RuntimeCmd: Adding entry to lpm match table ipv4_lpm
match key:           LPM-ff:ff:ff:ff/32
action:              broadcast
runtime data:        
Invalid table operation (DUPLICATE_ENTRY)
RuntimeCmd: Adding entry to exact match table forward
match key:           EXACT-ff:ff:ff:ff
action:              set_dmac
runtime data:        ff:ff:ff:ff:ff:ff
Invalid table operation (DUPLICATE_ENTRY)
RuntimeCmd: Creating multicast group 1
RuntimeCmd: Creating node with rid 0 , port map 1110 and lag map 
node was created with handle 1
RuntimeCmd: Associating node 0 to multicast group 1
Traceback (most recent call last):
  File "/usr/local/bin/simple_switch_CLI", line 31, in <module>
    sswitch_CLI.main()
  File "/usr/local/lib/python2.7/dist-packages/sswitch_CLI.py", line 93, in main
    SimpleSwitchAPI(args.pre, standard_client, mc_client, sswitch_client).cmdloop()
  File "/usr/lib/python2.7/cmd.py", line 142, in cmdloop
    stop = self.onecmd(line)
  File "/usr/lib/python2.7/cmd.py", line 221, in onecmd
    return func(arg)
  File "/usr/local/lib/python2.7/dist-packages/runtime_CLI.py", line 585, in handle
    return f(*args, **kwargs)
  File "/usr/local/lib/python2.7/dist-packages/runtime_CLI.py", line 1543, in do_mc_node_associate
    self.mc_client.bm_mc_node_associate(0, mgrp, l1_hdl)
  File "/usr/local/lib/python2.7/dist-packages/bm_runtime/simple_pre_lag/SimplePreLAG.py", line 222, in bm_mc_node_associate
    self.recv_bm_mc_node_associate()
  File "/usr/local/lib/python2.7/dist-packages/bm_runtime/simple_pre_lag/SimplePreLAG.py", line 246, in recv_bm_mc_node_associate
    raise result.ouch
bm_runtime.simple_pre_lag.ttypes.InvalidMcOperation: InvalidMcOperation(code=4)
**********
Network configuration for: h1
Default interface: h1-eth0      10.0.0.1        00:04:00:00:00:01
**********
**********
Network configuration for: h2
Default interface: h2-eth0      10.0.0.2        00:04:00:00:00:02
**********
**********
Network configuration for: h3
Default interface: h3-eth0      10.0.0.3        00:04:00:00:00:03
**********


h1 ./bcast_listen.py
h2 ./bcast_listen.py
h3 ./bcast_send.py
(None, '')
(None, "received 'hello everyone' from ('10.0.3.10', 60592)\n")
(None, "received 'hello everyone' from ('10.0.3.10', 60592)\n")
*** Stopping 0 controllers

*** Stopping 3 links
...
*** Stopping 1 switches
s1 
*** Stopping 3 hosts
h1 h2 h3 
*** Done
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ ls /tmp/p4app_logs/
h1.stdout     h2.stdout     h3.stdout     p4s.s1.log    s1-eth1.pcap  s1-eth2.pcap  s1-eth3.pcap  
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ cat /tmp/p4app_logs/p4s.s1.log   
Calling target program-options parser
[09:22:09.326] [bmv2] [D] [thread 83] Set default entry for table 'ipv4_lpm': NoAction - 
[09:22:09.326] [bmv2] [D] [thread 83] Set default entry for table 'forward': NoAction - 
[09:22:09.326] [bmv2] [D] [thread 83] Set default entry for table 'send_frame': NoAction - 
Adding interface s1-eth1 as port 1
[09:22:09.326] [bmv2] [D] [thread 83] Adding interface s1-eth1 as port 1
Adding interface s1-eth2 as port 2
[09:22:09.355] [bmv2] [D] [thread 83] Adding interface s1-eth2 as port 2
Adding interface s1-eth3 as port 3
[09:22:09.395] [bmv2] [D] [thread 83] Adding interface s1-eth3 as port 3
Thrift server was started
[09:22:10.462] [bmv2] [T] [thread 97] bm_get_config
[09:22:10.465] [bmv2] [T] [thread 97] bm_table_add_entry
[09:22:10.465] [bmv2] [D] [thread 97] Entry 0 added to table 'ipv4_lpm'
[09:22:10.465] [bmv2] [D] [thread 97] Dumping entry 0
Match key:
* ipv4.dstAddr        : LPM       ffffffff/32
Action entry: broadcast - 

[09:22:10.466] [bmv2] [T] [thread 97] bm_table_add_entry
[09:22:10.466] [bmv2] [D] [thread 97] Entry 0 added to table 'forward'
[09:22:10.466] [bmv2] [D] [thread 97] Dumping entry 0
Match key:
* ingress_metadata.nhop_ipv4: EXACT     ffffffff
Action entry: set_dmac - ffffffffffff,

[09:22:10.466] [bmv2] [T] [thread 97] bm_mc_mgrp_create
[09:22:10.466] [bmv2] [D] [thread 97] mgrp node created for mgid 1
[09:22:10.466] [bmv2] [T] [thread 97] bm_mc_node_create
[09:22:10.466] [bmv2] [D] [thread 97] node created for rid 0
[09:22:10.466] [bmv2] [T] [thread 97] bm_mc_node_associate
[09:22:10.466] [bmv2] [D] [thread 97] node associated with mgid 1
[09:22:10.467] [bmv2] [T] [thread 97] bm_table_add_entry
[09:22:10.467] [bmv2] [E] [thread 97] Error when trying to add entry to table 'ipv4_lpm'
[09:22:10.487] [bmv2] [T] [thread 97] bm_table_add_entry
[09:22:10.487] [bmv2] [E] [thread 97] Error when trying to add entry to table 'forward'
[09:22:10.488] [bmv2] [T] [thread 97] bm_mc_mgrp_create
[09:22:10.488] [bmv2] [D] [thread 97] mgrp node created for mgid 1
[09:22:10.488] [bmv2] [T] [thread 97] bm_mc_node_create
[09:22:10.488] [bmv2] [D] [thread 97] node created for rid 0
[09:22:10.488] [bmv2] [T] [thread 97] bm_mc_node_associate
[09:22:10.488] [bmv2] [E] [thread 97] node associate failed, node is already associated
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Processing packet received on port 3
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Parser 'parser': start
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Extracting header 'ethernet'
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Parser state 'start': key is 0800
[09:22:10.819] [bmv2] [T] [thread 89] [0.0] [cxt 0] Bytes parsed: 14
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Extracting header 'ipv4'
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Parser state 'parse_ipv4' has no switch, going to default next state
[09:22:10.819] [bmv2] [T] [thread 89] [0.0] [cxt 0] Bytes parsed: 34
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Parser 'parser': end
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Pipeline 'ingress': start
[09:22:10.819] [bmv2] [T] [thread 89] [0.0] [cxt 0] bcast_router.p4(76) Condition "hdr.ipv4.isValid()" is true
[09:22:10.819] [bmv2] [T] [thread 89] [0.0] [cxt 0] Applying table 'ipv4_lpm'
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Looking up key:
* ipv4.dstAddr        : ffffffff

[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Table 'ipv4_lpm': hit with handle 0
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Dumping entry 0
Match key:
* ipv4.dstAddr        : LPM       ffffffff/32
Action entry: broadcast - 

[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Action entry is broadcast - 
[09:22:10.819] [bmv2] [T] [thread 89] [0.0] [cxt 0] bcast_router.p4(42) Executing action broadcast
[09:22:10.819] [bmv2] [T] [thread 89] [0.0] [cxt 0] Applying table 'forward'
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Looking up key:
* ingress_metadata.nhop_ipv4: ffffffff

[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Table 'forward': hit with handle 0
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Dumping entry 0
Match key:
* ingress_metadata.nhop_ipv4: EXACT     ffffffff
Action entry: set_dmac - ffffffffffff,

[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Action entry is set_dmac - ffffffffffff,
[09:22:10.819] [bmv2] [T] [thread 89] [0.0] [cxt 0] bcast_router.p4(47) Executing action set_dmac
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Pipeline 'ingress': end
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Multicast requested for packet
[09:22:10.819] [bmv2] [D] [thread 89] number of packets replicated : 3
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Replicating packet on port 1
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Replicating packet on port 2
[09:22:10.819] [bmv2] [D] [thread 89] [0.0] [cxt 0] Replicating packet on port 3
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Pipeline 'egress': start
[09:22:10.820] [bmv2] [T] [thread 91] [0.1] [cxt 0] bcast_router.p4(27) Condition "hdr.ipv4.isValid()" is true
[09:22:10.820] [bmv2] [T] [thread 91] [0.1] [cxt 0] Applying table 'send_frame'
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Looking up key:
* standard_metadata.egress_port: 0001

[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Table 'send_frame': miss
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Action entry is NoAction - 
[09:22:10.820] [bmv2] [T] [thread 91] [0.1] [cxt 0] /usr/local/share/p4c/p4include/core.p4(55) Executing action NoAction
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Pipeline 'egress': end
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Deparser 'deparser': start
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Updating checksum 'cksum'
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Deparsing header 'ethernet'
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Deparsing header 'ipv4'
[09:22:10.820] [bmv2] [D] [thread 91] [0.1] [cxt 0] Deparser 'deparser': end
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Pipeline 'egress': start
[09:22:10.820] [bmv2] [T] [thread 92] [0.2] [cxt 0] bcast_router.p4(27) Condition "hdr.ipv4.isValid()" is true
[09:22:10.820] [bmv2] [T] [thread 92] [0.2] [cxt 0] Applying table 'send_frame'
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Looking up key:
* standard_metadata.egress_port: 0002

[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Table 'send_frame': miss
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Action entry is NoAction - 
[09:22:10.820] [bmv2] [T] [thread 92] [0.2] [cxt 0] /usr/local/share/p4c/p4include/core.p4(55) Executing action NoAction
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Pipeline 'egress': end
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Deparser 'deparser': start
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Updating checksum 'cksum'
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Deparsing header 'ethernet'
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Deparsing header 'ipv4'
[09:22:10.820] [bmv2] [D] [thread 92] [0.2] [cxt 0] Deparser 'deparser': end
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Pipeline 'egress': start
[09:22:10.820] [bmv2] [T] [thread 93] [0.3] [cxt 0] bcast_router.p4(27) Condition "hdr.ipv4.isValid()" is true
[09:22:10.820] [bmv2] [T] [thread 93] [0.3] [cxt 0] Applying table 'send_frame'
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Looking up key:
* standard_metadata.egress_port: 0003

[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Table 'send_frame': miss
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Action entry is NoAction - 
[09:22:10.820] [bmv2] [T] [thread 93] [0.3] [cxt 0] /usr/local/share/p4c/p4include/core.p4(55) Executing action NoAction
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Pipeline 'egress': end
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Deparser 'deparser': start
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Updating checksum 'cksum'
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Deparsing header 'ethernet'
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Deparsing header 'ipv4'
[09:22:10.820] [bmv2] [D] [thread 93] [0.3] [cxt 0] Deparser 'deparser': end
[09:22:10.820] [bmv2] [D] [thread 94] [0.1] [cxt 0] Transmitting packet of size 56 out of port 1
[09:22:10.820] [bmv2] [D] [thread 94] [0.2] [cxt 0] Transmitting packet of size 56 out of port 2
[09:22:10.820] [bmv2] [D] [thread 94] [0.3] [cxt 0] Transmitting packet of size 56 out of port 3
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples$ 
```

### docker 镜像大小

当前版本 docker 镜像只有 908MB，已集成 tshark / scapy 等工具，不算大。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
p4lang/p4app          latest              07024040be0e        6 hours ago         908MB
```
