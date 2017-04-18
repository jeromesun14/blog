title: 跑通 p4factory app int
date: 2017-04-18 16:45:20
toc: true
tags: [p4, int]
categories: p4
keywords: [p4, int, in band telemetry, factory, demo]
description: p4factory 样例 app int 跑通的过程记录。
---


vxlan 内核模块依赖内核 3.19

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

升级 Linux 内核到 3.19 后，docker 无法运行。

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
