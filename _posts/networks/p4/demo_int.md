title: 跑通 p4factory app int
date: 2017-04-18 16:45:20
toc: true
tags: [p4, int]
categories: p4
keywords: [p4, int, in band telemetry, p4factory, demo, ubuntu, 16.04, 3.19, kernel, linux, webui, success, 成功, docker, webui, iperf, failed]
description: p4factory 样例 app int 跑通的过程记录。
---

## 运行环境
ubuntu 16.04 + linux kernel 3.19

## 步骤
详见：

* [app/int/readme](https://github.com/p4lang/p4factory/tree/master/apps/int)
* [p4factory readme](https://github.com/p4lang/p4factory)
* [switch readme](https://github.com/p4lang/switch)

问题 fix 见下文。

## 运行成功图示

* 执行 `int_ref_topology.py`

![执行命令](/images/networks/p4/int_demo/run.png)

* iperf 打流

![打流](/images/networks/p4/int_demo/traffic.png)

* webui - 拓扑

![拓扑](/images/networks/p4/int_demo/webui.png)

* webui - flows

![multi-flow](/images/networks/p4/int_demo/multi-flow.png)

* webui - 时延

![latency](/images/networks/p4/int_demo/latency.png)

* webui - notification

![notifcation](/images/networks/p4/int_demo/notification.png)

## 问题解决记录
### 内核版本切换为 3.19

目前 vxlan-gpe 的内核模块基于内核 3.19，而目前常用 Linux 发行版的内核版本都在 4.2 以上，不切换则编译不过。**更好的办法是，直接使用内核版本为 3.19 的 Linux 发行版**。

详见 [p4factory issue 136](https://github.com/p4lang/p4factory/issues/136)
用于支持内核版本 3.19。详见另一篇文章 [ubuntu 用回旧版内核](http://sunyongfeng.com/201704/linux/startup_with_old_kernel.html)

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/apps/int/vxlan-gpe$ make
make -C /lib/modules/4.4.0-72-generic/build M=/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe modules
make[1]: Entering directory '/usr/src/linux-headers-4.4.0-72-generic'
  CC [M]  /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.o
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:94:7: error: redefinition of ‘union vxlan_addr’
 union vxlan_addr {
       ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:44:0:
include/net/vxlan.h:119:7: note: originally defined here
 union vxlan_addr {
       ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:100:8: error: redefinition of ‘struct vxlan_rdst’
 struct vxlan_rdst {
        ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:44:0:
include/net/vxlan.h:125:8: note: originally defined here
 struct vxlan_rdst {
        ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:122:8: error: redefinition of ‘struct vxlan_dev’
 struct vxlan_dev {
        ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:44:0:
include/net/vxlan.h:152:8: note: originally defined here
 struct vxlan_dev {
        ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: In function ‘vxlan_fdb_info’:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:423:9: error: void value not ignored as it ought to be
  return nlmsg_end(skb, nlh);
         ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: In function ‘vxlan_udp_encap_recv’:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1255:4: error: ‘struct vxlan_sock’ has no member named ‘rcv’
  vs->rcv(vs, skb, vxh->vx_vni);
    ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: In function ‘vxlan6_xmit_skb’:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1681:7: error: implicit declaration of function ‘vlan_tx_tag_present’ [-Werror=implicit-function-declaration]
    + (vlan_tx_tag_present(skb) ? VLAN_HLEN : 0);
       ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1708:23: warning:passing argument 1 of ‘udp_tunnel6_xmit_skb’ from incompatible pointer type [-Wincompatible-pointer-types]
  udp_tunnel6_xmit_skb(vs->sock, dst, skb, dev, saddr, daddr, prio,
                       ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:37:0:
include/net/udp_tunnel.h:87:5: note: expected ‘struct dst_entry *’ but argument is of type ‘struct socket *’
 int udp_tunnel6_xmit_skb(struct dst_entry *dst, struct sock *sk,
     ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1708:33: warning:passing argument 2 of ‘udp_tunnel6_xmit_skb’ from incompatible pointer type [-Wincompatible-pointer-types]
  udp_tunnel6_xmit_skb(vs->sock, dst, skb, dev, saddr, daddr, prio,
                                 ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:37:0:
include/net/udp_tunnel.h:87:5: note: expected ‘struct sock *’ but argument is of type ‘struct dst_entry *’
 int udp_tunnel6_xmit_skb(struct dst_entry *dst, struct sock *sk,
     ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1708:2: error: too few arguments to function ‘udp_tunnel6_xmit_skb’
  udp_tunnel6_xmit_skb(vs->sock, dst, skb, dev, saddr, daddr, prio,
  ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:37:0:
include/net/udp_tunnel.h:87:5: note: declared here
 int udp_tunnel6_xmit_skb(struct dst_entry *dst, struct sock *sk,
     ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: In function ‘vxlan_xmit_skb’:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1787:29: warning:passing argument 1 of ‘udp_tunnel_xmit_skb’ from incompatible pointer type [-Wincompatible-pointer-types]
  return udp_tunnel_xmit_skb(vs->sock, rt, skb, src, dst, tos,
                             ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:37:0:
include/net/udp_tunnel.h:81:5: note: expected ‘struct rtable *’ but argument is of type ‘struct socket *’
 int udp_tunnel_xmit_skb(struct rtable *rt, struct sock *sk, struct sk_buff *skb,
     ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1787:39: warning:passing argument 2 of ‘udp_tunnel_xmit_skb’ from incompatible pointer type [-Wincompatible-pointer-types]
  return udp_tunnel_xmit_skb(vs->sock, rt, skb, src, dst, tos,
                                       ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:37:0:
include/net/udp_tunnel.h:81:5: note: expected ‘struct sock *’ but argument is of type ‘struct rtable *’
 int udp_tunnel_xmit_skb(struct rtable *rt, struct sock *sk, struct sk_buff *skb,
     ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1787:9: error: too few arguments to function ‘udp_tunnel_xmit_skb’
  return udp_tunnel_xmit_skb(vs->sock, rt, skb, src, dst, tos,
         ^
In file included from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:37:0:
include/net/udp_tunnel.h:81:5: note: declared here
 int udp_tunnel_xmit_skb(struct rtable *rt, struct sock *sk, struct sk_buff *skb,
     ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: In function ‘vxlan_xmit_one’:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1941:34: warning:passing argument 1 of ‘ipv6_stub->ipv6_dst_lookup’ from incompatible pointer type [-Wincompatible-pointer-types]
   if (ipv6_stub->ipv6_dst_lookup(sk, &ndst, &fl6)) {
                                  ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1941:34: note: expected ‘struct net *’ but argument is of type ‘struct sock *’
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1941:38: warning:passing argument 2 of ‘ipv6_stub->ipv6_dst_lookup’ from incompatible pointer type [-Wincompatible-pointer-types]
   if (ipv6_stub->ipv6_dst_lookup(sk, &ndst, &fl6)) {
                                      ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1941:38: note: expected ‘struct sock *’ but argument is of type ‘struct dst_entry **’
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1941:45: warning:passing argument 3 of ‘ipv6_stub->ipv6_dst_lookup’ from incompatible pointer type [-Wincompatible-pointer-types]
   if (ipv6_stub->ipv6_dst_lookup(sk, &ndst, &fl6)) {
                                             ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1941:45: note: expected ‘struct dst_entry **’ but argument is of type ‘struct flowi6 *’
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1941:7: error: too few arguments to function ‘ipv6_stub->ipv6_dst_lookup’
   if (ipv6_stub->ipv6_dst_lookup(sk, &ndst, &fl6)) {
       ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: At top level:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2468:12: error: unknown type name ‘vxlan_rcv_t’
            vxlan_rcv_t *rcv, void *data,
            ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2520:7: error: unknown type name ‘vxlan_rcv_t’
       vxlan_rcv_t *rcv, void *data,
       ^
In file included from include/linux/linkage.h:6:0,
                 from include/linux/kernel.h:6,
                 from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:14:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2546:19: error: ‘vxlan_sock_add’ undeclared here (not in a function)
 EXPORT_SYMBOL_GPL(vxlan_sock_add);
                   ^
include/linux/export.h:57:16: note: in definition of macro ‘__EXPORT_SYMBOL’
  extern typeof(sym) sym;     \
                ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2546:1: note: in expansion of macro ‘EXPORT_SYMBOL_GPL’
 EXPORT_SYMBOL_GPL(vxlan_sock_add);
 ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: In function ‘vxlan_sock_work’:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2557:8: error: called object ‘vxlan_sock_add’ is not a function or function pointer
  nvs = vxlan_sock_add(net, port, vxlan_rcv, NULL, false, vxlan->flags);
        ^
In file included from include/linux/linkage.h:6:0,
                 from include/linux/kernel.h:6,
                 from /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:14:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2546:19: note: declared here
 EXPORT_SYMBOL_GPL(vxlan_sock_add);
                   ^
include/linux/export.h:57:21: note: in definition of macro ‘__EXPORT_SYMBOL’
  extern typeof(sym) sym;     \
                     ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2546:1: note: in expansion of macro ‘EXPORT_SYMBOL_GPL’
 EXPORT_SYMBOL_GPL(vxlan_sock_add);
 ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: At top level:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:604:25: warning: vxlan_gro_receive’ defined but not used [-Wunused-function]
 static struct sk_buff **vxlan_gro_receive(struct sk_buff **head, struct sk_buff *
                         ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:669:12: warning: vxlan_gro_complete’ defined but not used [-Wunused-function]
 static int vxlan_gro_complete(struct sk_buff *skb, int nhoff)
            ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:692:13: warning: vxlan_notify_add_rx_port’ defined but not used [-Wunused-function]
 static void vxlan_notify_add_rx_port(struct vxlan_sock *vs)
             ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1219:12: warning:‘vxlan_udp_encap_recv’ defined but not used [-Wunused-function]
 static int vxlan_udp_encap_recv(struct sock *sk, struct sk_buff *skb)
            ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2427:13: warning:‘vxlan_del_work’ defined but not used [-Wunused-function]
 static void vxlan_del_work(struct work_struct *work)
             ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:2434:23: warning:‘vxlan_create_sock’ defined but not used [-Wunused-function]
 static struct socket *vxlan_create_sock(struct net *net, bool ipv6,
                       ^
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c: In function ‘vxlan_xmit_skb’:
/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.c:1789:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
cc1: some warnings being treated as errors
scripts/Makefile.build:264: recipe for target '/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.o' failed
make[2]: *** [/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/vxlan.o] Error 1
Makefile:1420: recipe for target '_module_/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe' failed
make[1]: *** [_module_/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe] Error 2
make[1]: Leaving directory '/usr/src/linux-headers-4.4.0-72-generic'
Makefile:4: recipe for target 'all' failed
make: *** [all] Error 2
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/apps/int/vxlan-gpe$ make clean
make -C /lib/modules/4.4.0-72-generic/build M=/home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe clean
make[1]: Entering directory '/usr/src/linux-headers-4.4.0-72-generic'
  CLEAN   /home/sunyongfeng/workshop/p4factory/apps/int/vxlan-gpe/.tmp_versions
make[1]: Leaving directory '/usr/src/linux-headers-4.4.0-72-generic'
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/apps/int/vxlan-gpe$

```

### 解决 Linux 内核到 3.19 后，docker 无法运行

详见另一篇文章 [Linux升级内核后 docker 不可用](http://sunyongfeng.com/201704/linux/docker_failed_after_update_kernel.html)

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ sudo service docker start
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ sudo systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since 二 2017-04-18 16:38:26 CST; 9s ago
     Docs: https://docs.docker.com
  Process: 3058 ExecStart=/usr/bin/dockerd -H fd:// (code=exited, status=1/FAILURE)
 Main PID: 3058 (code=exited, status=1/FAILURE)

4月 18 16:38:25 openswitch-OptiPlex-380 systemd[1]: Starting Docker Application Container Engine...
4月 18 16:38:25 openswitch-OptiPlex-380 dockerd[3058]: time="2017-04-18T16:38:25.914144290+08:00" level=info msg="libcontainerd: new containerd process, pid: 3064"
4月 18 16:38:26 openswitch-OptiPlex-380 dockerd[3058]: time="2017-04-18T16:38:26.962727971+08:00" level=error msg="[graphdriver] prior storage driver \"aufs\" failed:
4月 18 16:38:26 openswitch-OptiPlex-380 dockerd[3058]: time="2017-04-18T16:38:26.962806596+08:00" level=fatal msg="Error starting daemon: error initializing graphdriv
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: Failed to start Docker Application Container Engine.
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: docker.service: Unit entered failed state.
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: docker.service: Failed with result 'exit-code'.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ sudo systemctl start docker
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ docker --version
Docker version 1.12.3, build 6b644ec
```

### 解决 int_ref_topology.py 失败问题

#### docker_node.py 运行失败
详见 [p4factory issue 193](https://github.com/p4lang/p4factory/issues/193)

执行 mininet 起 docker 起不来。[解决](http://stackoverflow.com/questions/28006027/valueerror-invalid-literal-for-int-with-base-10-n):

> replace :
> 
> `start = int(line)` 
> 
> to
> 
> `start = int(line.strip())   # strip will chop the '\n' `
> 
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$ sudo ./int_ref_topology.py --model-dir=$P4HOME/install               
Adding switch spine1
Traceback (most recent call last):
  File "./int_ref_topology.py", line 131, in <module>
    run_cfg(model_dir)
  File "./int_ref_topology.py", line 91, in run_cfg
    net = mgr.setupAndStartNetwork()
  File "/home/sunyongfeng/workshop/p4factory/mininet/int_cfg.py", line 140, in setupAndStartNetwork
    self.addSwitches()
  File "/home/sunyongfeng/workshop/p4factory/mininet/int_cfg.py", line 177, in addSwitches
    pps = s.pps, qdepth = s.qdepth )
  File "/usr/lib/python2.7/dist-packages/mininet/net.py", line 240, in addSwitch
    sw = cls( name, **defaults )
  File "/home/sunyongfeng/workshop/p4factory/mininet/docker/p4model.py", line 55, in __init__
    DockerSwitch.__init__( self, name, **kwargs )
  File "/home/sunyongfeng/workshop/p4factory/mininet/docker/docker_node.py", line 30, in __init__
    Node.__init__( self, name, **kwargs )
  File "/usr/lib/python2.7/dist-packages/mininet/node.py", line 106, in __init__
    self.startShell()
  File "/home/sunyongfeng/workshop/p4factory/mininet/docker/docker_node.py", line 119, in startShell
    self.pid = int(ps_out[0])
ValueError: invalid literal for int() with base 10: "'21431'\n"
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$
```

ps_out 的结果 `["'22326'\n"]`，因此 L119 应改为：

```
self.pid = int((ps_out[0].strip()).strip('\''))
```

> [链接](http://www.runoob.com/python/att-string-strip.html)：Python strip() 方法用于移除字符串头尾指定的字符（默认为空格）。

#### 解决 docker 内 bmv2 无法运行的问题
非必需。

原因：ubuntu 16.04 默认 gcc 版为 5.4.0，而 bmv2 docker ubuntu 版本为 14.04，其默认 gcc 为 4.8。因此升级 docker 中的 ubuntu 版本为 16.04

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$ sudo ./int_ref_topology.py --model-dir=$P4HOME/install               
Adding switch spine1
Adding switch spine2
Adding switch leaf1
Adding switch leaf2

Waiting 10 seconds for switches to intialize...
INT Config  spine1
Traceback (most recent call last):
  File "./int_ref_topology.py", line 131, in <module>
    run_cfg(model_dir)
  File "./int_ref_topology.py", line 91, in run_cfg
    net = mgr.setupAndStartNetwork()
  File "/home/sunyongfeng/workshop/p4factory/mininet/int_cfg.py", line 147, in setupAndStartNetwork
    self.configSwitches()
  File "/home/sunyongfeng/workshop/p4factory/mininet/int_cfg.py", line 237, in configSwitches
    self.configSwitch(s)
  File "/home/sunyongfeng/workshop/p4factory/mininet/int_cfg.py", line 251, in configSwitch
    client.switcht_api_init( device )
  File "/home/sunyongfeng/workshop/p4factory/submodules/switch/switchapi/switch_api_thrift/switch_api_rpc.py", line 1565, in switcht_api_init
    return self.recv_switcht_api_init()
  File "/home/sunyongfeng/workshop/p4factory/submodules/switch/switchapi/switch_api_thrift/switch_api_rpc.py", line 1577, in recv_switcht_api_init
    (fname, mtype, rseqid) = iprot.readMessageBegin()
  File "/usr/local/lib/python2.7/dist-packages/thrift/protocol/TBinaryProtocol.py", line 134, in readMessageBegin
    sz = self.readI32()
  File "/usr/local/lib/python2.7/dist-packages/thrift/protocol/TBinaryProtocol.py", line 217, in readI32
    buff = self.trans.readAll(4)
  File "/usr/local/lib/python2.7/dist-packages/thrift/transport/TTransport.py", line 60, in readAll
    chunk = self.read(sz - have)
  File "/usr/local/lib/python2.7/dist-packages/thrift/transport/TTransport.py", line 161, in read
    self.__rbuf = BufferIO(self.__trans.read(max(sz, self.__rbuf_size)))
  File "/usr/local/lib/python2.7/dist-packages/thrift/transport/TSocket.py", line 117, in read
    buff = self.handle.recv(sz)
socket.error: [Errno 104] Connection reset by peer
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$ 
```

根本原因为工具链问题，gcc 版本不一致。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                               NAMES
50b20ec98aac        p4dockerswitch_bmv2   "/bin/sh -c /bin/bash"   47 seconds ago      Up 45 seconds       0.0.0.0:26001->9091/tcp, 0.0.0.0:27001->10001/tcp   leaf2
d04a024c559e        p4dockerswitch_bmv2   "/bin/sh -c /bin/bash"   48 seconds ago      Up 46 seconds       0.0.0.0:26000->9091/tcp, 0.0.0.0:27000->10001/tcp   leaf1
4678cbfe6b2b        p4dockerswitch_bmv2   "/bin/sh -c /bin/bash"   49 seconds ago      Up 47 seconds       0.0.0.0:26003->9091/tcp, 0.0.0.0:27003->10001/tcp   spine2
701da5748dc9        p4dockerswitch_bmv2   "/bin/sh -c /bin/bash"   50 seconds ago      Up 48 seconds       0.0.0.0:26002->9091/tcp, 0.0.0.0:27002->10001/tcp   spine1
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$ docker attach 701da5748dc9
bin      dev   lib    mnt                   nnpy       proc  sbin  third-party          tmp
boot     etc   lib64  nanomsg-1.0.0         opt        root  srv   thrift-0.9.2         usr
configs  home  media  nanomsg-1.0.0.tar.gz  p4factory  run   sys   thrift-0.9.2.tar.gz  var
bmv2_driver.log  bmv2_model.log  start.sh
cat: /tmp/dmvdr: No such file or directory
bin      dev   lib    mnt                   nnpy       proc  sbin  third-party          tmp
boot     etc   lib64  nanomsg-1.0.0         opt        root  srv   thrift-0.9.2         usr
configs  home  media  nanomsg-1.0.0.tar.gz  p4factory  run   sys   thrift-0.9.2.tar.gz  var
bmv2_driver.log  bmv2_model.log  start.sh
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by /home/sunyongfeng/workshop/p4/install/lib/bmpd/switch/libpd.so.0)
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `CXXABI_1.3.8' not found (required by /home/sunyongfeng/workshop/p4/install/lib/bmpd/switch/libpd.so.0)
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by /home/sunyongfeng/workshop/p4/install/lib/bmpd/switch/libpd.so.0)
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by /home/sunyongfeng/workshop/p4/install/lib/bmpd/switch/libpdthrift.so.0)
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by /home/sunyongfeng/workshop/p4/install/lib/libbmpdfixed.so.0)
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by /home/sunyongfeng/workshop/p4/install/lib/libbmpdfixedthrift.so.0)
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by /home/sunyongfeng/workshop/p4/install/lib/libruntimestubs.so.0)
/home/sunyongfeng/workshop/p4/install/bin/bmswitchp4_drivers: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by /home/sunyongfeng/workshop/p4/install/lib/libsimpleswitch_thrift.so.0)
/home/sunyongfeng/workshop/p4/install/bin/simple_switch: error while loading shared libraries: libboost_system.so.1.58.0: cannot open shared object file: No such file or directory
```

更新后的 Dockerfile patch:

```patch
diff --git a/docker/Dockerfile b/docker/Dockerfile
old mode 100644
new mode 100755
index 42c87af..e9003fc
--- a/docker/Dockerfile
+++ b/docker/Dockerfile
@@ -1,4 +1,4 @@
-FROM      ubuntu:14.04
+FROM      ubuntu:16.04
 MAINTAINER Antonin Bas <antonin@barefootnetworks.com>
 
 RUN apt-get update
@@ -41,7 +41,6 @@ RUN apt-get install -y \
     redis-server \
     redis-tools \
     subversion \
-    tshark \
     xterm
 
 RUN pip install tenjin
@@ -84,7 +83,7 @@ RUN mkdir -p /tmp/install_tmp ; \
     wget -c http://archive.apache.org/dist/thrift/0.9.2/thrift-0.9.2.tar.gz ; \
     tar zxvf thrift-0.9.2.tar.gz ; \
     cd thrift-0.9.2 ; \
-    ./configure --with-cpp=yes --with-c_glib=no --with-java=no --with-ruby=no --with-erlang=no --with-go=no --with-nodejs=no ; \
+    ./configure ; \
     make -j4 ; \
     make install ; \
     ldconfig ; \
```

#### DockerFile 解决 tshark 卡在 yes/no

DockerFile 的改造，tshark 无法直接配置通过，卡在 yes/no。详见上一小节 Dockerfile，先不安装 tshark，等 bmv2 docker image 打好后，再进去安装、重新 commit docker image。

#### 在 docker 内使用 simple_switch_CLI

非必需，仅用于确认表项是否正常下发。

leaf/spine docker 内无法使用 simple_switch_CLI：

* 改 Dockerfile 的 thrift 配置为默认配置，不简化 docker 内的 thrift
* 在 docker 内通过 pip 安装 thrift，`pip install --upgrade thrift`
* 通过命令访问 simple_switch_CLI，`/home/sunyongfeng/workshop/p4/install/bin/simple_switch_CLI --json /home/sunyongfeng/workshop/p4/install/share/bmpd/switch/switch.json --thrift-port 10001`

没有安装好 thrift 的问题：

```
root@leaf1:/home/sunyongfeng# /home/sunyongfeng/workshop/p4/install/bin/simple_switch_CLI
Traceback (most recent call last):
  File "/home/sunyongfeng/workshop/p4/install/bin/simple_switch_CLI", line 30, in <module>
    import sswitch_CLI
  File "/home/sunyongfeng/workshop/p4/install/lib/python2.7/site-packages/sswitch_CLI.py", line 23, in <module>
    import runtime_CLI
  File "/home/sunyongfeng/workshop/p4/install/lib/python2.7/site-packages/runtime_CLI.py", line 30, in <module>
    import bmpy_utils as utils
  File "/home/sunyongfeng/workshop/p4/install/lib/python2.7/site-packages/bmpy_utils.py", line 30, in <module>
    from thrift.protocol import TMultiplexedProtocol
ImportError: cannot import name TMultiplexedProtocol
```

一开始不知道 simple_switch thrift 开放的端口是多少，通过 netstat -anp 确认为 10001。

```
root@leaf1:/# /home/sunyongfeng/workshop/p4/install/bin/simple_switch_CLI --json /home/sunyongfeng/workshop/p4/install/share/bmpd/switch/switch.json 
Error when requesting config md5 sum from switch
root@leaf1:/# netstat -anp                                                  
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:9090            0.0.0.0:*               LISTEN      81/bmswitchp4_drive
tcp        0      0 0.0.0.0:9091            0.0.0.0:*               LISTEN      81/bmswitchp4_drive
tcp        0      0 0.0.0.0:9092            0.0.0.0:*               LISTEN      81/bmswitchp4_drive
tcp        0      0 127.0.0.1:2601          0.0.0.0:*               LISTEN      114/zebra       
tcp        0      0 127.0.0.1:2605          0.0.0.0:*               LISTEN      118/bgpd        
tcp        0      0 0.0.0.0:10001           0.0.0.0:*               LISTEN      67/simple_switch
tcp        0      0 0.0.0.0:179             0.0.0.0:*               LISTEN      118/bgpd        
tcp        0      0 127.0.0.1:10001         127.0.0.1:40983         ESTABLISHED 67/simple_switch
tcp        0      0 127.0.0.1:40984         127.0.0.1:10001         ESTABLISHED 81/bmswitchp4_drive
tcp        0      0 127.0.0.1:10001         127.0.0.1:40985         ESTABLISHED 67/simple_switch
tcp        0      0 127.0.0.1:10001         127.0.0.1:40984         ESTABLISHED 67/simple_switch
tcp        0      0 127.0.0.1:40983         127.0.0.1:10001         ESTABLISHED 81/bmswitchp4_drive
tcp        0      0 127.0.0.1:9090          127.0.0.1:46833         TIME_WAIT   -               
tcp        0      0 127.0.0.1:40985         127.0.0.1:10001         ESTABLISHED 81/bmswitchp4_drive
tcp        0      0 127.0.0.1:9090          127.0.0.1:46829         TIME_WAIT   -               
tcp6       0      0 :::179                  :::*                    LISTEN      118/bgpd        
raw6       0      0 :::58                   :::*                    7           114/zebra       
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   PID/Program name    Path
unix  2      [ ACC ]     STREAM     LISTENING     576586   118/bgpd            /var/run/quagga/bgpd.vty
unix  2      [ ACC ]     STREAM     LISTENING     575606   114/zebra           /var/run/quagga/zserv.api
unix  2      [ ACC ]     STREAM     LISTENING     575609   114/zebra           /var/run/quagga/zebra.vty
unix  2      [ ACC ]     STREAM     LISTENING     574079   67/simple_switch    /tmp/bmv2-0-notifications.ipc
unix  3      [ ]         STREAM     CONNECTED     574211   81/bmswitchp4_drive 
unix  3      [ ]         STREAM     CONNECTED     575248   67/simple_switch    
unix  3      [ ]         STREAM     CONNECTED     576590   118/bgpd            
unix  3      [ ]         STREAM     CONNECTED     574212   81/bmswitchp4_drive 
unix  3      [ ]         STREAM     CONNECTED     574218   67/simple_switch    /tmp/bmv2-0-notifications.ipc
unix  3      [ ]         STREAM     CONNECTED     575295   81/bmswitchp4_drive 
unix  3      [ ]         STREAM     CONNECTED     575841   123/watchquagga     
unix  3      [ ]         STREAM     CONNECTED     575271   81/bmswitchp4_drive 
unix  3      [ ]         STREAM     CONNECTED     576591   114/zebra           /var/run/quagga/zserv.api
unix  3      [ ]         STREAM     CONNECTED     576807   118/bgpd            /var/run/quagga/bgpd.vty
unix  3      [ ]         STREAM     CONNECTED     578621   114/zebra           /var/run/quagga/zebra.vty
unix  3      [ ]         STREAM     CONNECTED     575296   81/bmswitchp4_drive 
unix  3      [ ]         STREAM     CONNECTED     576589   114/zebra           /var/run/quagga/zserv.api
unix  3      [ ]         STREAM     CONNECTED     576588   118/bgpd            
unix  3      [ ]         STREAM     CONNECTED     575247   67/simple_switch    
unix  3      [ ]         STREAM     CONNECTED     574344   81/bmswitchp4_drive 
unix  3      [ ]         STREAM     CONNECTED     574345   81/bmswitchp4_drive 
unix  3      [ ]         STREAM     CONNECTED     578620   123/watchquagga     
root@leaf1:/# /home/sunyongfeng/workshop/p4/install/bin/simple_switch_CLI --json /home/sunyongfeng/workshop/p4/install/share/bmpd/switch/switch.json --thrift-port 10001
Control utility for runtime P4 table manipulation
RuntimeCmd:
```

查看 `ipv4_fib` 和 `ipv4_fib_lpm` 表，确认表项安装 OK。

```
RuntimeCmd: show_tables
acl_stats                      [implementation=None, mk=]
adjust_lkp_fields              [implementation=None, mk=ipv4_valid(valid, 1),   ipv6_valid(valid, 1)]
bd_flood                       [implementation=None, mk=ingress_metadata.bd(exact, 16), l2_metadata.lkp_pkt_type(exact, 3)]
compute_ipv4_hashes            [implementation=None, mk=ingress_metadata.drop_flag(exact, 1)]
compute_ipv6_hashes            [implementation=None, mk=ingress_metadata.drop_flag(exact, 1)]
compute_non_ip_hashes          [implementation=None, mk=ingress_metadata.drop_flag(exact, 1)]
compute_other_hashes           [implementation=None, mk=hash_metadata.hash1(exact, 16)]
dmac                           [implementation=None, mk=ingress_metadata.bd(exact, 16), l2_metadata.lkp_mac_da(exact, 48)]
drop_stats                     [implementation=None, mk=]
ecmp_group                     [implementation=ecmp_action_profile, mk=l3_metadata.nexthop_index(exact, 16)]
egress_bd_map                  [implementation=None, mk=egress_metadata.bd(exact, 16)]
egress_bd_stats                [implementation=None, mk=egress_metadata.bd(exact, 16),  l2_metadata.lkp_pkt_type(exact, 3)]
egress_filter                  [implementation=None, mk=]
egress_filter_drop             [implementation=None, mk=]
egress_ip_acl                  [implementation=None, mk=acl_metadata.egress_if_label(ternary, 16),      acl_metadata.egress_bd_label(ternary, 16),      ipv4.srcAddr(ternary, 32),   ipv4.dstAddr(ternary, 32),      ipv4.protocol(ternary, 8),      acl_metadata.egress_src_port_range_id(exact, 8),        acl_metadata.egress_dst_port_range_id(exact, 8)]
egress_ipv6_acl                [implementation=None, mk=acl_metadata.egress_if_label(ternary, 16),      acl_metadata.egress_bd_label(ternary, 16),      ipv6.srcAddr(ternary, 128),  ipv6.dstAddr(ternary, 128),     ipv6.nextHdr(ternary, 8),       acl_metadata.egress_src_port_range_id(exact, 8),        acl_metadata.egress_dst_port_range_id(exact, 8)]
egress_l4_dst_port             [implementation=None, mk=l3_metadata.egress_l4_dport(range, 16)]
egress_l4_src_port             [implementation=None, mk=l3_metadata.egress_l4_sport(range, 16)]
egress_l4port_fields           [implementation=None, mk=tcp_valid(valid, 1),    udp_valid(valid, 1),    icmp_valid(valid, 1)]
egress_mac_acl                 [implementation=None, mk=acl_metadata.egress_if_label(ternary, 16),      acl_metadata.egress_bd_label(ternary, 16),      ethernet.srcAddr(ternary, 48),       ethernet.dstAddr(ternary, 48),  ethernet.etherType(ternary, 16)]
egress_nat                     [implementation=None, mk=nat_metadata.nat_rewrite_index(exact, 14)]
egress_port_mapping            [implementation=None, mk=standard_metadata.egress_port(exact, 9)]
egress_qos_map                 [implementation=None, mk=qos_metadata.egress_qos_group(ternary, 5),      qos_metadata.lkp_tc(ternary, 8)]
egress_system_acl              [implementation=None, mk=fabric_metadata.reason_code(ternary, 16),       standard_metadata.egress_port(ternary, 9),      intrinsic_metadata.deflection_flag(ternary, 1),      l3_metadata.l3_mtu_check(ternary, 16),  acl_metadata.acl_deny(ternary, 1)]
egress_vlan_xlate              [implementation=None, mk=egress_metadata.ifindex(exact, 16),     egress_metadata.bd(exact, 16)]
egress_vni                     [implementation=None, mk=egress_metadata.bd(exact, 16),  tunnel_metadata.egress_tunnel_type(exact, 5)]
fabric_ingress_dst_lkp         [implementation=None, mk=fabric_header.dstDevice(exact, 8)]
fabric_ingress_src_lkp         [implementation=None, mk=fabric_header_multicast.ingressIfindex(exact, 16)]
fabric_lag                     [implementation=fabric_lag_action_profile, mk=fabric_metadata.dst_device(exact, 8)]
fwd_result                     [implementation=None, mk=l2_metadata.l2_redirect(ternary, 1),    acl_metadata.acl_redirect(ternary, 1),  acl_metadata.racl_redirect(ternary, 1),      l3_metadata.rmac_hit(ternary, 1),       l3_metadata.fib_hit(ternary, 1),        nat_metadata.nat_hit(ternary, 1),       l2_metadata.lkp_pkt_type(ternary, 3),        l3_metadata.lkp_ip_type(ternary, 2),    multicast_metadata.igmp_snooping_enabled(ternary, 1),   multicast_metadata.mld_snooping_enabled(ternary, 1),multicast_metadata.mcast_route_hit(ternary, 1),  multicast_metadata.mcast_bridge_hit(ternary, 1),        multicast_metadata.mcast_rpf_group(ternary, 16),        multicast_metadata.mcast_mode(ternary, 2)]
ingress_bd_stats               [implementation=None, mk=]
ingress_l4_dst_port            [implementation=None, mk=l3_metadata.lkp_l4_dport(range, 16)]
ingress_l4_src_port            [implementation=None, mk=l3_metadata.lkp_l4_sport(range, 16)]
ingress_port_mapping           [implementation=None, mk=standard_metadata.ingress_port(exact, 9)]
ingress_port_properties        [implementation=None, mk=standard_metadata.ingress_port(exact, 9)]
ingress_qos_map_dscp           [implementation=None, mk=qos_metadata.ingress_qos_group(ternary, 5),     l3_metadata.lkp_dscp(ternary, 8)]
ingress_qos_map_pcp            [implementation=None, mk=qos_metadata.ingress_qos_group(ternary, 5),     l2_metadata.lkp_pcp(ternary, 3)]
int_bos                        [implementation=None, mk=int_header.total_hop_cnt(ternary, 8),   int_header.instruction_mask_0003(ternary, 4),   int_header.instruction_mask_0407(ternary, 4),        int_header.instruction_mask_0811(ternary, 4),   int_header.instruction_mask_1215(ternary, 4)]
int_insert                     [implementation=None, mk=int_metadata_i2e.source(ternary, 1),    int_metadata_i2e.sink(ternary, 1),      int_header_valid(valid, 1)]
int_inst_0003                  [implementation=None, mk=int_header.instruction_mask_0003(exact, 4)]
int_inst_0407                  [implementation=None, mk=int_header.instruction_mask_0407(exact, 4)]
int_inst_0811                  [implementation=None, mk=int_header.instruction_mask_0811(exact, 4)]
int_inst_1215                  [implementation=None, mk=int_header.instruction_mask_1215(exact, 4)]
int_meta_header_update         [implementation=None, mk=int_metadata.insert_cnt(ternary, 8)]
int_outer_encap                [implementation=None, mk=ipv4_valid(valid, 1),   vxlan_gpe_valid(valid, 1),      int_metadata_i2e.source(exact, 1),      tunnel_metadata.egress_tunnel_type(ternary, 5)]
int_sink_update_outer          [implementation=None, mk=vxlan_gpe_int_header_valid(valid, 1),   ipv4_valid(valid, 1),   int_metadata_i2e.sink(exact, 1)]
int_source                     [implementation=None, mk=int_header_valid(valid, 1),     ipv4_valid(valid, 1),   ipv4_metadata.lkp_ipv4_da(ternary, 32), ipv4_metadata.lkp_ipv4_sa(ternary, 32),      inner_ipv4_valid(valid, 1),     inner_ipv4.dstAddr(ternary, 32),        inner_ipv4.srcAddr(ternary, 32)]
int_terminate                  [implementation=None, mk=int_header_valid(valid, 1),     vxlan_gpe_int_header_valid(valid, 1),   ipv4_valid(valid, 1),   ipv4_metadata.lkp_ipv4_da(ternary, 32),      inner_ipv4_valid(valid, 1),     inner_ipv4.dstAddr(ternary, 32)]
ip_acl                         [implementation=None, mk=acl_metadata.if_label(ternary, 16),     acl_metadata.bd_label(ternary, 16),     ipv4_metadata.lkp_ipv4_sa(ternary, 32),      ipv4_metadata.lkp_ipv4_da(ternary, 32), l3_metadata.lkp_ip_proto(ternary, 8),   acl_metadata.ingress_src_port_range_id(exact, 8),       acl_metadata.ingress_dst_port_range_id(exact, 8),    tcp.flags(ternary, 8),  l3_metadata.lkp_ip_ttl(ternary, 8)]
ipsg                           [implementation=None, mk=ingress_metadata.ifindex(exact, 16),    ingress_metadata.bd(exact, 16), l2_metadata.lkp_mac_sa(exact, 48),  ipv4_metadata.lkp_ipv4_sa(exact, 32)]
ipsg_permit_special            [implementation=None, mk=l3_metadata.lkp_ip_proto(ternary, 8),   l3_metadata.lkp_l4_dport(ternary, 16),  ipv4_metadata.lkp_ipv4_da(ternary, 32)]
ipv4_dest_vtep                 [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4.dstAddr(exact, 32),        tunnel_metadata.ingress_tunnel_type(exact, 5)]
ipv4_fib                       [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_da(exact, 32)]
ipv4_fib_lpm                   [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_da(lpm, 32)]
ipv4_multicast_bridge          [implementation=None, mk=ingress_metadata.bd(exact, 16), ipv4_metadata.lkp_ipv4_sa(exact, 32),   ipv4_metadata.lkp_ipv4_da(exact, 32)]
ipv4_multicast_bridge_star_g   [implementation=None, mk=ingress_metadata.bd(exact, 16), ipv4_metadata.lkp_ipv4_da(exact, 32)]
ipv4_multicast_route           [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_sa(exact, 32),   ipv4_metadata.lkp_ipv4_da(exact, 32)]
ipv4_multicast_route_star_g    [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_da(exact, 32)]
ipv4_racl                      [implementation=None, mk=acl_metadata.bd_label(ternary, 16),     ipv4_metadata.lkp_ipv4_sa(ternary, 32), ipv4_metadata.lkp_ipv4_da(ternary, 32),      l3_metadata.lkp_ip_proto(ternary, 8),   acl_metadata.ingress_src_port_range_id(exact, 8),       acl_metadata.ingress_dst_port_range_id(exact, 8)]
ipv4_src_vtep                  [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4.srcAddr(exact, 32),        tunnel_metadata.ingress_tunnel_type(exact, 5)]
ipv4_urpf                      [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_sa(exact, 32)]
ipv4_urpf_lpm                  [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_sa(lpm, 32)]
ipv6_acl                       [implementation=None, mk=acl_metadata.if_label(ternary, 16),     acl_metadata.bd_label(ternary, 16),     ipv6_metadata.lkp_ipv6_sa(ternary, 128),     ipv6_metadata.lkp_ipv6_da(ternary, 128),        l3_metadata.lkp_ip_proto(ternary, 8),   acl_metadata.ingress_src_port_range_id(exact, 8),       acl_metadata.ingress_dst_port_range_id(exact, 8),    tcp.flags(ternary, 8),  l3_metadata.lkp_ip_ttl(ternary, 8)]
ipv6_dest_vtep                 [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6.dstAddr(exact, 128),       tunnel_metadata.ingress_tunnel_type(exact, 5)]
ipv6_fib                       [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6_metadata.lkp_ipv6_da(exact, 128)]
ipv6_fib_lpm                   [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6_metadata.lkp_ipv6_da(lpm, 128)]
ipv6_multicast_bridge          [implementation=None, mk=ingress_metadata.bd(exact, 16), ipv6_metadata.lkp_ipv6_sa(exact, 128),  ipv6_metadata.lkp_ipv6_da(exact, 128)]
ipv6_multicast_bridge_star_g   [implementation=None, mk=ingress_metadata.bd(exact, 16), ipv6_metadata.lkp_ipv6_da(exact, 128)]
ipv6_multicast_route           [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6_metadata.lkp_ipv6_sa(exact, 128),  ipv6_metadata.lkp_ipv6_da(exact, 128)]
ipv6_multicast_route_star_g    [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6_metadata.lkp_ipv6_da(exact, 128)]
ipv6_racl                      [implementation=None, mk=acl_metadata.bd_label(ternary, 16),     ipv6_metadata.lkp_ipv6_sa(ternary, 128),        ipv6_metadata.lkp_ipv6_da(ternary, 128),     l3_metadata.lkp_ip_proto(ternary, 8),   acl_metadata.ingress_src_port_range_id(exact, 8),       acl_metadata.ingress_dst_port_range_id(exact, 8)]
ipv6_src_vtep                  [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6.srcAddr(exact, 128),       tunnel_metadata.ingress_tunnel_type(exact, 5)]
ipv6_urpf                      [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6_metadata.lkp_ipv6_sa(exact, 128)]
ipv6_urpf_lpm                  [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv6_metadata.lkp_ipv6_sa(lpm, 128)]
l3_rewrite                     [implementation=None, mk=ipv4_valid(valid, 1),   ipv6_valid(valid, 1),   mpls[0]_valid(valid, 1),        ipv4.dstAddr(ternary, 32),  ipv6.dstAddr(ternary, 128)]
lag_group                      [implementation=lag_action_profile, mk=ingress_metadata.egress_ifindex(exact, 16)]
learn_notify                   [implementation=None, mk=l2_metadata.l2_src_miss(ternary, 1),    l2_metadata.l2_src_move(ternary, 16),   l2_metadata.stp_state(ternary, 3)]
mac_acl                        [implementation=None, mk=acl_metadata.if_label(ternary, 16),     acl_metadata.bd_label(ternary, 16),     l2_metadata.lkp_mac_sa(ternary, 48), l2_metadata.lkp_mac_da(ternary, 48),    l2_metadata.lkp_mac_type(ternary, 16)]
meter_action                   [implementation=None, mk=meter_metadata.packet_color(exact, 2),  meter_metadata.meter_index(exact, 16)]
meter_index                    [implementation=None, mk=meter_metadata.meter_index(exact, 16)]
mirror                         [implementation=None, mk=i2e_metadata.mirror_session_id(exact, 16)]
mpls                           [implementation=None, mk=tunnel_metadata.mpls_label(exact, 20),  inner_ipv4_valid(valid, 1),     inner_ipv6_valid(valid, 1)]
mtu                            [implementation=None, mk=l3_metadata.mtu_index(exact, 8),        ipv4_valid(valid, 1),   ipv6_valid(valid, 1)]
nat_dst                        [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_da(exact, 32),   l3_metadata.lkp_ip_proto(exact, 8), l3_metadata.lkp_l4_dport(exact, 16)]
nat_flow                       [implementation=None, mk=l3_metadata.vrf(ternary, 16),   ipv4_metadata.lkp_ipv4_sa(ternary, 32), ipv4_metadata.lkp_ipv4_da(ternary, 32),      l3_metadata.lkp_ip_proto(ternary, 8),   l3_metadata.lkp_l4_sport(ternary, 16),  l3_metadata.lkp_l4_dport(ternary, 16)]
nat_src                        [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_sa(exact, 32),   l3_metadata.lkp_ip_proto(exact, 8), l3_metadata.lkp_l4_sport(exact, 16)]
nat_twice                      [implementation=None, mk=l3_metadata.vrf(exact, 16),     ipv4_metadata.lkp_ipv4_sa(exact, 32),   ipv4_metadata.lkp_ipv4_da(exact, 32)l3_metadata.lkp_ip_proto(exact, 8),      l3_metadata.lkp_l4_sport(exact, 16),    l3_metadata.lkp_l4_dport(exact, 16)]
native_packet_over_fabric      [implementation=None, mk=ipv4_valid(valid, 1),   ipv6_valid(valid, 1)]
nexthop                        [implementation=None, mk=l3_metadata.nexthop_index(exact, 16)]
outer_ipv4_multicast           [implementation=None, mk=multicast_metadata.ipv4_mcast_key_type(exact, 1),       multicast_metadata.ipv4_mcast_key(exact, 16),   ipv4.srcAddr(exact, 32),     ipv4.dstAddr(exact, 32)]
outer_ipv4_multicast_star_g    [implementation=None, mk=multicast_metadata.ipv4_mcast_key_type(exact, 1),       multicast_metadata.ipv4_mcast_key(exact, 16),   ipv4.dstAddr(ternary, 32)]
outer_ipv6_multicast           [implementation=None, mk=multicast_metadata.ipv6_mcast_key_type(exact, 1),       multicast_metadata.ipv6_mcast_key(exact, 16),   ipv6.srcAddr(exact, 128),    ipv6.dstAddr(exact, 128)]
outer_ipv6_multicast_star_g    [implementation=None, mk=multicast_metadata.ipv6_mcast_key_type(exact, 1),       multicast_metadata.ipv6_mcast_key(exact, 16),   ipv6.dstAddr(ternary, 128)]
outer_rmac                     [implementation=None, mk=l3_metadata.rmac_group(exact, 10),      ethernet.dstAddr(exact, 48)]
port_vlan_mapping              [implementation=bd_action_profile, mk=ingress_metadata.ifindex(exact, 16),       vlan_tag_[0]_valid(valid, 1),   vlan_tag_[0].vid(exact, 12), vlan_tag_[1]_valid(valid, 1),   vlan_tag_[1].vid(exact, 12)]
replica_type                   [implementation=None, mk=multicast_metadata.replica(exact, 1),   egress_metadata.same_bd_check(ternary, 16)]
rewrite                        [implementation=None, mk=l3_metadata.nexthop_index(exact, 16)]
rewrite_multicast              [implementation=None, mk=ipv4_valid(valid, 1),   ipv6_valid(valid, 1),   ipv4.dstAddr(ternary, 32),      ipv6.dstAddr(ternary, 128)]
rid                            [implementation=None, mk=intrinsic_metadata.egress_rid(exact, 16)]
rmac                           [implementation=None, mk=l3_metadata.rmac_group(exact, 10),      l2_metadata.lkp_mac_da(exact, 48)]
sflow_ing_take_sample          [implementation=None, mk=ingress_metadata.sflow_take_sample(ternary, 32),        sflow_metadata.sflow_session_id(exact, 16)]
sflow_ingress                  [implementation=None, mk=ingress_metadata.ifindex(ternary, 16),  ipv4_metadata.lkp_ipv4_sa(ternary, 32), ipv4_metadata.lkp_ipv4_da(ternary, 32),      sflow_valid(valid, 1)]
smac                           [implementation=None, mk=ingress_metadata.bd(exact, 16), l2_metadata.lkp_mac_sa(exact, 48)]
smac_rewrite                   [implementation=None, mk=egress_metadata.smac_idx(exact, 9)]
spanning_tree                  [implementation=None, mk=ingress_metadata.ifindex(exact, 16),    l2_metadata.stp_group(exact, 10)]
storm_control                  [implementation=None, mk=standard_metadata.ingress_port(exact, 9),       l2_metadata.lkp_pkt_type(ternary, 3)]
storm_control_stats            [implementation=None, mk=meter_metadata.packet_color(exact, 2),  standard_metadata.ingress_port(exact, 9)]
switch_config_params           [implementation=None, mk=]
system_acl                     [implementation=None, mk=acl_metadata.if_label(ternary, 16),     acl_metadata.bd_label(ternary, 16),     ingress_metadata.ifindex(ternary, 16),       l2_metadata.lkp_mac_type(ternary, 16),  l2_metadata.port_vlan_mapping_miss(ternary, 1), security_metadata.ipsg_check_fail(ternary, 1),  acl_metadata.acl_deny(ternary, 1),   acl_metadata.racl_deny(ternary, 1),     l3_metadata.urpf_check_fail(ternary, 1),        ingress_metadata.drop_flag(ternary, 1), l3_metadata.l3_copy(ternary, 1),     l3_metadata.rmac_hit(ternary, 1),       l3_metadata.routed(ternary, 1), ipv6_metadata.ipv6_src_is_link_local(ternary, 1),       l2_metadata.same_if_check(ternary, 16),      tunnel_metadata.tunnel_if_check(ternary, 1),    l3_metadata.same_bd_check(ternary, 16), l3_metadata.lkp_ip_ttl(ternary, 8),     l2_metadata.stp_state(ternary, 3),   ingress_metadata.control_frame(ternary, 1),     ipv4_metadata.ipv4_unicast_enabled(ternary, 1), ipv6_metadata.ipv6_unicast_enabled(ternary, 1),      ingress_metadata.egress_ifindex(ternary, 16),   fabric_metadata.reason_code(ternary, 16)]
traffic_class                  [implementation=None, mk=qos_metadata.tc_qos_group(ternary, 5),  qos_metadata.lkp_tc(ternary, 8)]
tunnel                         [implementation=None, mk=tunnel_metadata.tunnel_vni(exact, 24),  tunnel_metadata.ingress_tunnel_type(exact, 5),  inner_ipv4_valid(valid, 1),  inner_ipv6_valid(valid, 1)]
tunnel_decap_process_inner     [implementation=None, mk=inner_tcp_valid(valid, 1),      inner_udp_valid(valid, 1),      inner_icmp_valid(valid, 1)]
tunnel_decap_process_outer     [implementation=None, mk=tunnel_metadata.ingress_tunnel_type(exact, 5),  inner_ipv4_valid(valid, 1),     inner_ipv6_valid(valid, 1)]
tunnel_dmac_rewrite            [implementation=None, mk=tunnel_metadata.tunnel_dmac_index(exact, 14)]
tunnel_dst_rewrite             [implementation=None, mk=tunnel_metadata.tunnel_dst_index(exact, 14)]
tunnel_encap_process_inner     [implementation=None, mk=ipv4_valid(valid, 1),   ipv6_valid(valid, 1),   tcp_valid(valid, 1),    udp_valid(valid, 1),    icmp_valid(valid, 1)]
tunnel_encap_process_outer     [implementation=None, mk=tunnel_metadata.egress_tunnel_type(exact, 5),   tunnel_metadata.egress_header_count(exact, 4),  multicast_metadata.replica(exact, 1)]
tunnel_lookup_miss             [implementation=None, mk=ipv4_valid(valid, 1),   ipv6_valid(valid, 1)]
tunnel_mtu                     [implementation=None, mk=tunnel_metadata.tunnel_index(exact, 14)]
tunnel_rewrite                 [implementation=None, mk=tunnel_metadata.tunnel_index(exact, 14)]
tunnel_smac_rewrite            [implementation=None, mk=tunnel_metadata.tunnel_smac_index(exact, 9)]
tunnel_src_rewrite             [implementation=None, mk=tunnel_metadata.tunnel_src_index(exact, 9)]
urpf_bd                        [implementation=None, mk=l3_metadata.urpf_bd_group(exact, 16),   ingress_metadata.bd(exact, 16)]
validate_mpls_packet           [implementation=None, mk=mpls[0].label(ternary, 20),     mpls[0].bos(ternary, 1),        mpls[0]_valid(valid, 1),        mpls[1].label(ternary, 20),  mpls[1].bos(ternary, 1),        mpls[1]_valid(valid, 1),        mpls[2].label(ternary, 20),     mpls[2].bos(ternary, 1),        mpls[2]_valid(valid, 1)]
validate_outer_ethernet        [implementation=None, mk=ethernet.srcAddr(ternary, 48),  ethernet.dstAddr(ternary, 48),  vlan_tag_[0]_valid(valid, 1),   vlan_tag_[1]_valid(valid, 1)]
validate_outer_ipv4_packet     [implementation=None, mk=ipv4.version(ternary, 4),       ipv4.ttl(ternary, 8),   ipv4.srcAddr(ternary, 32)]
validate_outer_ipv6_packet     [implementation=None, mk=ipv6.version(ternary, 4),       ipv6.hopLimit(ternary, 8),      ipv6.srcAddr(ternary, 128)]
validate_packet                [implementation=None, mk=l2_metadata.lkp_mac_sa(ternary, 48),    l2_metadata.lkp_mac_da(ternary, 48),    l3_metadata.lkp_ip_type(ternary, 2), l3_metadata.lkp_ip_ttl(ternary, 8),     l3_metadata.lkp_ip_version(ternary, 4), ipv4_metadata.lkp_ipv4_sa(ternary, 32), ipv6_metadata.lkp_ipv6_sa(ternary, 128)]
vlan_decap                     [implementation=None, mk=vlan_tag_[0]_valid(valid, 1),   vlan_tag_[1]_valid(valid, 1)]
RuntimeCmd: table_dump ipv4_fib
==========
TABLE ENTRIES
**********
Dumping entry 0x0
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     00000000
Action entry: fib_hit_nexthop - 06
**********
Dumping entry 0x1
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     0a000164
Action entry: fib_hit_nexthop - 03
**********
Dumping entry 0x2
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     0a000264
Action entry: fib_hit_nexthop - 03
**********
Dumping entry 0x3
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     0a010b01
Action entry: fib_hit_nexthop - 03
**********
Dumping entry 0x4
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     0a010c01
Action entry: fib_hit_nexthop - 03
**********
Dumping entry 0x1000005
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     0a000202
Action entry: fib_hit_nexthop - 07
**********
Dumping entry 0x3000006
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     0a010c02
Action entry: fib_hit_nexthop - 08
**********
Dumping entry 0x7
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: EXACT     0a000101
Action entry: fib_hit_nexthop - 09
==========
Dumping default entry
Action entry: on_miss - 
==========
RuntimeCmd: table_dump ipv4_fib_lpm
==========
TABLE ENTRIES
**********
Dumping entry 0x0
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: LPM       7f000000/8
Action entry: fib_hit_nexthop - 05
**********
Dumping entry 0x1
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: LPM       00000000/0
Action entry: fib_hit_nexthop - 06
**********
Dumping entry 0x2
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: LPM       0a000164/24
Action entry: fib_hit_nexthop - 03
**********
Dumping entry 0x3
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: LPM       0a000264/24
Action entry: fib_hit_nexthop - 03
**********
Dumping entry 0x4
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: LPM       0a010b01/24
Action entry: fib_hit_nexthop - 03
**********
Dumping entry 0x5
Match key:
* l3_metadata.vrf          : EXACT     0001
* ipv4_metadata.lkp_ipv4_da: LPM       0a010c01/24
Action entry: fib_hit_nexthop - 03
==========
Dumping default entry
Action entry: on_miss - 
==========
RuntimeCmd: 
root@leaf1:/# 
```

### 解决 mininet 起来后 host 间无法互 Ping

原因：docker 的物理端口配置问题，实际拓扑使用 leaf1-eth1，但是配置却配到 swp1。

详见 [p4factory issue 194](https://github.com/p4lang/p4factory/issues/194)

patch 如下：

```patch
diff --git a/mininet/configs/leaf1/l3_int_ref_topo/startup_config.sh b/mininet/configs/leaf1/l3_int_ref_topo/startup_config.sh
index 6432a4c..5d34e05 100755
--- a/mininet/configs/leaf1/l3_int_ref_topo/startup_config.sh
+++ b/mininet/configs/leaf1/l3_int_ref_topo/startup_config.sh
@@ -2,15 +2,15 @@
 
 stty -echo; set +m
 
-ip link set dev swp1 address 00:01:00:00:00:01
-ip link set dev swp2 address 00:01:00:00:00:02
-ip link set dev swp3 address 00:01:00:00:00:03
-ip link set dev swp4 address 00:01:00:00:00:04
+ip link set dev leaf1-eth1 address 00:01:00:00:00:01
+ip link set dev leaf1-eth2 address 00:01:00:00:00:02
+ip link set dev leaf1-eth3 address 00:01:00:00:00:03
+ip link set dev leaf1-eth4 address 00:01:00:00:00:04
 
-ip address add 10.0.1.100/24 broadcast + dev swp1
-ip address add 10.0.2.100/24 broadcast + dev swp2
-ip address add 10.1.11.1/24 broadcast + dev swp3
-ip address add 10.1.12.1/24 broadcast + dev swp4
+ip address add 10.0.1.100/24 broadcast + dev leaf1-eth1
+ip address add 10.0.2.100/24 broadcast + dev leaf1-eth2
+ip address add 10.1.11.1/24 broadcast + dev leaf1-eth3
+ip address add 10.1.12.1/24 broadcast + dev leaf1-eth4
 
 cp /configs/quagga/* /etc/quagga/
 chown quagga.quagga /etc/quagga/*
diff --git a/mininet/configs/leaf2/l3_int_ref_topo/startup_config.sh b/mininet/configs/leaf2/l3_int_ref_topo/startup_config.sh
index 0a5637f..7562c72 100755
--- a/mininet/configs/leaf2/l3_int_ref_topo/startup_config.sh
+++ b/mininet/configs/leaf2/l3_int_ref_topo/startup_config.sh
@@ -2,15 +2,15 @@
 
 stty -echo; set +m
 
-ip link set dev swp1 address 00:02:00:00:00:01
-ip link set dev swp2 address 00:02:00:00:00:02
-ip link set dev swp3 address 00:02:00:00:00:03
-ip link set dev swp4 address 00:02:00:00:00:04
+ip link set dev leaf2-eth1 address 00:02:00:00:00:01
+ip link set dev leaf2-eth2 address 00:02:00:00:00:02
+ip link set dev leaf2-eth3 address 00:02:00:00:00:03
+ip link set dev leaf2-eth4 address 00:02:00:00:00:04
 
-ip address add 10.0.3.100/24 broadcast + dev swp1
-ip address add 10.0.4.100/24 broadcast + dev swp2
-ip address add 10.1.21.1/24 broadcast + dev swp3
-ip address add 10.1.22.1/24 broadcast + dev swp4
+ip address add 10.0.3.100/24 broadcast + dev leaf2-eth1
+ip address add 10.0.4.100/24 broadcast + dev leaf2-eth2
+ip address add 10.1.21.1/24 broadcast + dev leaf2-eth3
+ip address add 10.1.22.1/24 broadcast + dev leaf2-eth4
 
 cp /configs/quagga/* /etc/quagga/
 chown quagga.quagga /etc/quagga/*
diff --git a/mininet/configs/spine1/l3_int_ref_topo/startup_config.sh b/mininet/configs/spine1/l3_int_ref_topo/startup_config.sh
index f2ecfc0..33f4878 100755
--- a/mininet/configs/spine1/l3_int_ref_topo/startup_config.sh
+++ b/mininet/configs/spine1/l3_int_ref_topo/startup_config.sh
@@ -2,11 +2,11 @@
 
 stty -echo; set +m
 
-ip link set dev swp1 address 00:03:00:00:00:01
-ip link set dev swp2 address 00:03:00:00:00:02
+ip link set dev spine1-eth1 address 00:03:00:00:00:01
+ip link set dev spine1-eth2 address 00:03:00:00:00:02
 
-ip address add 10.1.11.2/24 broadcast + dev swp1
-ip address add 10.1.21.2/24 broadcast + dev swp2
+ip address add 10.1.11.2/24 broadcast + dev spine1-eth1
+ip address add 10.1.21.2/24 broadcast + dev spine2-eth2
 
 cp /configs/quagga/* /etc/quagga/
 chown quagga.quagga /etc/quagga/*
diff --git a/mininet/configs/spine2/l3_int_ref_topo/startup_config.sh b/mininet/configs/spine2/l3_int_ref_topo/startup_config.sh
index 7a28663..be95656 100755
--- a/mininet/configs/spine2/l3_int_ref_topo/startup_config.sh
+++ b/mininet/configs/spine2/l3_int_ref_topo/startup_config.sh
@@ -2,11 +2,11 @@
 
 stty -echo; set +m
 
-ip link set dev swp1 address 00:04:00:00:00:01
-ip link set dev swp2 address 00:04:00:00:00:02
+ip link set dev spine2-eth1 address 00:04:00:00:00:01
+ip link set dev spine2-eth2 address 00:04:00:00:00:02
 
-ip address add 10.1.12.2/24 broadcast + dev swp1
-ip address add 10.1.22.2/24 broadcast + dev swp2
+ip address add 10.1.12.2/24 broadcast + dev spine2-eth1
+ip address add 10.1.22.2/24 broadcast + dev spine2-eth2
 
 cp /configs/quagga/* /etc/quagga/
 chown quagga.quagga /etc/quagga/*
```

### iperf 提示拒绝连接
非必需，仅出现过一次，默认每个 host 的 iperf server 都会启动。
h3 iperf 似乎默认没有启动。

```
 root  ~  workshop  p4factory  mininet  iperf -c 10.2.1.3 -t 60
connect failed: Connection refused
```

在 h3 中启动：

```
 root  ~  workshop  p4factory  mininet  iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[ 18] local 10.2.1.3 port 5001 connected with 10.2.1.1 port 45594
```

在 h1 中 运行 iperf 客户端：

```
 root  ~  workshop  p4factory  mininet  iperf -c 10.2.1.3 -t 6000000
------------------------------------------------------------
Client connecting to 10.2.1.3, TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[ 17] local 10.2.1.1 port 45594 connected with 10.2.1.3 port 5001
^C[ ID] Interval       Transfer     Bandwidth
[ 17]  0.0-639.1 sec   341 MBytes  4.47 Mbits/sec
```

### monitor 网页无法显示正在运行的流

目前在 h1 iperf h3 的时候，可以通过在 leaf1 中 `tshark -i leaf1-eth1` 确认输入报文，`tshark -i leaf1-eth4` 确认输出报文。
可看到报文是 vxlan-gpe 封装（UDP port 4790），但是目前 monitor client 网页上还无法看到 INT 汇总会的图表以及正在运行的流。

* 问题原因：bridge 过滤掉 UDP 报文，具体为哪条过滤表项目前还未细究。

> Okay. The solution to this lies in the quirks of how linux bridges get filtered. Essentially, you need to go in disable all the filters so that the UDP packets can get through. Details here: https://wiki.linuxfoundation.org/networking/bridge

* 解决方法：

```
# cd /proc/sys/net/bridge
# for f in bridge-nf-*; do echo 0 > $f; done
```

详见 p4-dev 邮件列表，[INT demo issue - Bad UDP checksum for packets headed	to monitor](http://lists.p4.org/pipermail/p4-dev_lists.p4.org/2017-May/001061.html)。
