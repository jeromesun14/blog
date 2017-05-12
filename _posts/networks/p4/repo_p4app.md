title: p4app 仓库 —— 用于快速验证 P4 程序
date: 2017-05-10 16:41:00
toc: true
tags: [p4]
categories: p4
keywords: [p4, p4app, 快速验证, p4lang, github, repo, topo, stf, bmv2, build, compile, docker, mininet]
description: p4app 仓库使用说明。
---

p4app
=====

## 简介
2017-02-23 p4lang 新建 [p4app](https://github.com/p4lang/p4app) 仓库，用于快速、简便地构建、运行、调试和测试 P4 程序。摆脱 [p4factory](https://github.com/p4lang/p4factory) / [switch](https://github.com/p4lang/switch) 依赖安装步骤复杂、编译时间长的问题。降低入门门槛。与预期的类似，p4app 将 bmv2 等基础组件打成 docker image。

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

p4app 不仅支持 mininet，还支持一种名为 `stf` （simple testing framework）的模拟网络测试框架。本文称 Mininet 或 stf 为后端（backend）， p4app 支持一个程序多个后端，在 `p4app.json` 文件中体现，并称其为 target，即 target 与 backend 实例的关系是 1:N。

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

每个 p4app 包包含一个 P4 程序，但可以包含多个 backend。例如，一个 p4app 可能包含多个不同的 Mininet 网络不配置，一组完整的 STF 测试套件，或上述两种的混合化。p4app 运行默认 backend，若想运行非默认 backend，则在运行 p4app 时以该 backend 名为参数。

```
p4app run examples/simple_router.p4app mininet

```

差不多就这样了！但还有一个有用的命令，p4app在本地缓存P4编译器和工具，因此您不必每次都重新下载。如果需要更新到最新版本，运行：

```
p4app update
```

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

样例 manifest 告诉 p4app 应该运行  `my_program.p4`，该 p4 程序使用 `p4-14` 编写（你同样也可以用 `p4-16`，P4-16 草案版本）。该样例定义一个 backend `mininet`（在 `p4app.json` 中 backend 称为 target），同时提供一些 Mininet 配置选项：测试网络有两个 host，一个模拟交换机，该交换机启机默认加载 `my_program.config` 中的配置。如果你在 `p4app.json` 中引用像 `my_program.config` 时，则需要将 `my_program.config` 文件包在 p4app 包中，p4app 将确保相应工具可找到它。

如果有多个 backend，且使用者没有通过名字指定默认 backend，则 p4app 随机运行其中一个。可使用 `default-target` 选项，设置默认 backend。例如：

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

另外，如果想与他人分享 p4app 包，运行  `p4app pack my-program.p4app` 压缩 p4app 包为单独文件。p4app 可以直接运行这个包，无需解压。如果想看解压包中的文件，运行 `p4app unpack my-p4ogram.p4app` ，p4app 将该文件解压回目录。


Backends
========

mininet
-------

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

BMV2 debugger 很方便，可在[这里](https://github.com/p4lang/behavioral-model/blob/master/docs/p4dbg_user_guide.md)阅读如何使用。

该 backend 还支持`compile-bmv2` 配置，详见下文相应章节。

multiswitch
-----------
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

该配置将创建这样的拓扑：

```
h1 <---> s1 <---> s2 <---> h2
```

其中 `s2-h2` 链路人工配置 50ms 的延迟。host 配置选项：

 - `cmd` - 在 host 上运行的命令
 - `wait` - 等待命令执行结束。如果配置成 false，表示在后台运行此命令。
 - `startup_sleep` - 启动命令后应等待的时间（以秒为单位）。
 - `latency` - 主机与交换机之间的延迟。配置值是数字（解释为秒）或具有时间单位的字符串（例如`50ms`或`1s`）。该配置将覆盖“links”对象中设置的延迟。

通过将主机名（例如“h1”）替换为相应的IP地址来格式化该命令。 目标中指定的参数将作为环境变量（即`$`后跟变量名称）可用于命令。例如：



The command is formatted by replacing the hostnames (e.g. `h1`) with the
corresponding IP address. The parameters specified in the target will be
available to the command as environment variables (i.e. `$` followed by the variable name). For an example, have a look at the [manifest](examples/multiswitch.p4app/p4app.json) for the multiswitch example app.

#### Limitations
Currently, each host can be connected to at most one switch.


#### Specifying entries for each switch
The routing tables (`ipv4_lpm`, `send_frame` and `forward`) are automatically
populated with this target. Additionally, you can specify custom entries to be
added to each switch. You can either include a commands file, or an array of
entries. These custom entries will have precedence over the automatically
generated ones for the routing tables. For example:

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

If the entries for `s2` above overlap with the automatically generated entries
(e.g. there is an automatic entry for `set_nhop 10.0.1.10/32`), these custom
entries will have precedence and you will see a warning about a duplicate entry
while the tables are being populated.

#### Custom topology class
Instead of letting this target create the mininet Topo class, you can use your
own. Specify the name of your module with the `topo_module` option. For example:

```
"multiswitch": {
  ...
  "topo_module": "mytopo"
  ...
}
```

This will import the `mytopo` module, `mytopo.py`, which should be in the same
directory as the manifest file (`p4app.json`). The module should implement the
class `CustomAppTopo`. It can extend the default topo class,
[apptopo.AppTopo](docker/scripts/mininet/apptopo.py).  For example:

```
# mytopo.py
from apptopo import AppTopo

class CustomAppTopo(AppTopo):
    def __init__(self, *args, **kwargs):
        AppTopo.__init__(self, *args, **kwargs)

        print self.links()
```

See the [customtopo.p4app](examples/customtopo.p4app/mytopo.py) working example.

#### Custom controller
Similarly to the `topo_module` option, you can specify a controller with the
`controller_module` option. This module should implement the class
`CustomAppController`. The default controller class is
[appcontroller.AppController](docker/scripts/mininet/appcontroller.py). You can
extend this class, as shown in the
[customtopo.p4app](examples/customtopo.p4app/mycontroller.py) example.


#### Logging
When this target is run, a temporary directory on the host, `/tmp/p4app_log`,
is mounted on the guest at `/tmp/p4app_log`. All data in this directory is
persisted to the host after running the p4app. The stdout from the hosts'
commands is stored in this location. If you need to save the output (e.g. logs)
of a command, you can also put that in this directory.

To save the debug logs from the P4 switches, set `"bmv2_log": true` in the
target. To capture PCAPs from all switches, set `"pcap_dump": true`. These
files will be saved to `/tmp/p4app_log`. For example usage, see the
[manifest](examples/broadcast.p4app/p4app.json) for the broadcast example app.

#### Cleanup commands
If you need to execute commands in the docker container after running the
target (and before mininet is stopped), you can use `after`. `after` should
contain `cmd`, which can either be a command or a list of commands. For
example:

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

custom
-----------

This is a third method for compiling a P4 program to run in a Mininet
environment. This target allows you to specify a Python `program` that
uses Mininet's Python API to specify the network topology and configuration.
For example:

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

This target will invoke the python script `topo.py` to start Mininet.
The `program` will be called with the following arguments:

| Argument         | Description |
| --------         | ----------- |
| --behavioral-exe | Value will be the switch executable |
| --json           | Value will be the P4 compiler output |
| --cli            | Value will be the switch command line interface program |

Example invocation:

```
PYTHONPATH=$PYTHONPATH:/scripts/mininet/ python2 topo.py \
                        --behavioral-exe simple_switch \
                        --json SOME_FILE \
                        --cli simple_switch_CLI
```

You can specify additional arguments to pass to your custom topology program by
including them in your `program` definition as follows:

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

The `program` can find the docker container ID in the `HOSTNAME` environment
variable so that it can output useful commands for copy/paste:

```python
import os
container = os.environ['HOSTNAME']
print 'Run the switch CLI as follows:'
print '  docker exec -t -i %s %s' % (container, args.cli)
```

stf
---

This target compiles the provided P4 program and run a test against it written
for the STF testing framework.

There is one configuration value, which is required:

```
"stf": {
  "test": "file.stf"
}
```

You must write the file specified by `test` in the STF format, which is
unfortunately currently undocumented. (If you'd like to reverse engineer it and
provide some documentation, please submit a PR!) You can take a look at the
example p4apps included in this repo to get a sense of the basics.

This target also supports the configuration values for the `compile-bvm2` target.

compile-bmv2
------------

This is a simple backend that just attempts to compile the provided P4 program
for the BMV2 architecture.

The following optional configuration values are supported:

```
"compile-bmv2": {
  "compiler-flags": ["-v", "-E"],
  "run-before-compile": ["date"],
  "run-after-compile": ["date"]
}
```

Advanced Features
=================

If you're hacking on the P4 toolchain or p4app itself, you may want to use a
modified Docker image instead of the standard p4lang one. That's easy to do;
just set the `P4APP_IMAGE` environment variable to the Docker image you'd like
to use. For example:

```
P4APP_IMAGE=me/my_p4app_image:latest p4app run examples/simple_router.p4app
```

#### Specify the name of the manifest file
By default, p4app will use the manifest file called `p4app.json` in the app's
directory. If your manifest file is not called `p4app.json`, you can use the
`--manifest` option to specify the name of the manifest. For example:

```
p4app run myapp.p4app --manifest testing.p4app
```

#### Specify location of log directory
By default, p4app will mount the directory `/tmp/p4app_logs` on the host to
`/tmp/p4app_logs` on the docker container guest. The output from bmv2, as well
as any output from your programs, will be saved to this directory.  Instead of
using the default directory (`/tmp/p4app_logs`), you can specify another
directory with the `$P4APP_LOGDIR` environment variable. For example, if you
run:

```
P4APP_LOGDIR=./out p4app run myapp.p4app
```

all the log files will be stored to `./out`.**

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

### docker 镜像大小

```
docker 镜像目前只有 908MB，已集成 tshark / scapy 等工具，不算大。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
p4lang/p4app          latest              07024040be0e        6 hours ago         908MB
```
