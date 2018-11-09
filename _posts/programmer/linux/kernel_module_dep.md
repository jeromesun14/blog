title: linux kernel module 依赖相关
date: 2017-06-22 16:58:00
toc: true
tags: [kernel]
categories: kernel
keywords: [Linux, kernel, module, dependency, 依赖, 关系, 内核, 模块]
description: linux 内核模块依赖关系查看方法，及依赖关系相关的问题汇总。
---

## 查看依赖关系

问题背景：从专有系统移植某个特定功能的内核模块到 Linux 发行版，确认该模块的依赖关系。包含“我依赖谁”、“谁依赖我”两种情况。
详见[How to check kernel module dependencies on Linux](http://xmodulo.com/how-to-check-kernel-module-dependencies-on-linux.html)

* lsmod

如下，vboxdrv.ko 依赖 vboxnetadp.ko、vboxnetflt.ko、vboxpci.ko。

```
sunyongfeng@ubuntu:~$ lsmod
Module                  Size  Used by
tipc                  127285  0
pci_stub               12622  1
vboxpci                23116  0
vboxnetadp             25813  0
vboxnetflt             27947  0
vboxdrv               455053  3 vboxnetadp,vboxnetflt,vboxpci
xt_CHECKSUM            12549  1
iptable_mangle         12734  1
ipt_REJECT             12541  2
nf_reject_ipv4         13183  1 ipt_REJECT
ebtable_filter         12827  0
ebtables               35359  1 ebtable_filter
ip6table_filter        12815  0
ip6_tables             27504  1 ip6table_filter
kvm_intel             154139  0
binfmt_misc            18163  1
cfg80211              551242  0
sch_netem              17419  0
vxlan                  41123  0
ip6_udp_tunnel         12755  1 vxlan
udp_tunnel             13187  1 vxlan
sch_htb                22429  0
xt_nat                 12726  0
xt_tcpudp              12924  6
btrfs                 968059  0
xor                    21411  1 btrfs
raid6_pq               97812  1 btrfs
ufs                    75315  0
qnx4                   13394  0
hfsplus               103788  0
hfs                    54819  0
minix                  36454  0
ntfs                  101958  0
msdos                  17332  0
jfs                   186807  0
xfs                   937845  0
veth                   13376  0
ipt_MASQUERADE         12678  4
nf_nat_masquerade_ipv4    13412  1 ipt_MASQUERADE
xfrm_user              36115  1
xfrm_algo              15394  1 xfrm_user
iptable_nat            12875  1
nf_conntrack_ipv4      18953  3
nf_defrag_ipv4         12758  1 nf_conntrack_ipv4
nf_nat_ipv4            14267  1 iptable_nat
xt_addrtype            12713  2
iptable_filter         12810  1
ip_tables              27718  3 iptable_filter,iptable_mangle,iptable_nat
xt_conntrack           12760  2
x_tables               34103  13 ip6table_filter,xt_CHECKSUM,ip_tables,xt_tcpudp,ipt_MASQUERADE,xt_conntrack,xt_nat,iptable_filter,ebtables,ipt_REJECT,iptable_mangle,ip6_tables,xt_addrtype
nf_nat                 26308  3 nf_nat_ipv4,xt_nat,nf_nat_masquerade_ipv4
nf_conntrack          105683  5 nf_nat,nf_nat_ipv4,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_ipv4
br_netfilter           18017  0
bridge                114479  1 br_netfilter
stp                    12976  1 bridge
llc                    14441  2 stp,bridge
overlay                42380  0
openvswitch            85378  0
libcrc32c              12644  2 xfs,openvswitch
snd_hda_codec_realtek    80584  1
snd_hda_codec_generic    70069  1 snd_hda_codec_realtek
snd_hda_intel          30775  6
dell_wmi               13132  0
snd_hda_controller     35493  1 snd_hda_intel
sparse_keymap          13890  1 dell_wmi
gpio_ich               13636  0
snd_hda_codec         144641  4 snd_hda_codec_realtek,snd_hda_codec_generic,snd_hda_intel,snd_hda_controller
snd_hwdep              17709  1 snd_hda_codec
coretemp               13638  0
snd_pcm               106401  3 snd_hda_codec,snd_hda_intel,snd_hda_controller
snd_seq_midi           13564  0
snd_seq_midi_event     14899  1 snd_seq_midi
kvm                   480880  1 kvm_intel
snd_rawmidi            31148  1 snd_seq_midi
snd_seq                63540  2 snd_seq_midi_event,snd_seq_midi
snd_seq_device         14875  3 snd_seq,snd_rawmidi,snd_seq_midi
snd_timer              30069  2 snd_pcm,snd_seq
dcdbas                 15017  0
lpc_ich                21176  0
snd                    83976  22 snd_hda_codec_realtek,snd_hwdep,snd_timer,snd_pcm,snd_seq,snd_rawmidi,snd_hda_codec_generic,snd_hda_codec,snd_hda_intel,snd_seq_device
serio_raw              13434  0
soundcore              15091  2 snd,snd_hda_codec
8250_fintek            12924  0
shpchp                 37216  0
mac_hid                13275  0
parport_pc             32909  1
ppdev                  17711  0
lp                     17799  0
parport                42432  3 lp,ppdev,parport_pc
autofs4                39306  2
hid_generic            12559  0
psmouse               118539  0
broadcom               17339  0
i915                 1087204  3
pata_acpi              13053  0
usbhid                 53106  0
hid                   110883  2 hid_generic,usbhid
wmi                    19379  1 dell_wmi
video                  24803  1 i915
i2c_algo_bit           13564  1 i915
drm_kms_helper        123797  1 i915
tg3                   179117  0
ptp                    19534  1 tg3
pps_core               19332  1 ptp
drm                   341489  5 i915,drm_kms_helper
```

* cat modules.dep，几乎与 lsmod 一样，但是包含未加载的内核模块依赖关系，ubuntu 默认有很多很多！

```
sunyongfeng@ubuntu:~$ cat /lib/modules/`uname -r`/modules.dep
kernel/arch/x86/kernel/cpu/mcheck/mce-inject.ko:
kernel/arch/x86/kernel/msr.ko:
kernel/arch/x86/kernel/cpuid.ko:
kernel/arch/x86/kernel/iosf_mbi.ko:
kernel/arch/x86/crypto/glue_helper.ko:
kernel/arch/x86/crypto/aes-x86_64.ko:
kernel/arch/x86/crypto/des3_ede-x86_64.ko: kernel/crypto/des_generic.ko
kernel/arch/x86/crypto/camellia-x86_64.ko: kernel/crypto/xts.ko kernel/crypto/lrw.ko kernel/crypto/gf128mul.ko kernel/arch/x86/crypto/glue_helper.ko
kernel/arch/x86/crypto/blowfish-x86_64.ko: kernel/crypto/blowfish_common.ko
kernel/arch/x86/crypto/twofish-x86_64.ko: kernel/crypto/twofish_common.ko
kernel/arch/x86/crypto/twofish-x86_64-3way.ko: kernel/arch/x86/crypto/twofish-x86_64.ko kernel/crypto/twofish_common.ko kernel/crypto/xts.ko kernel/crypto/lrw.ko kernel/crypto/gf128mul.ko kernel/arch/x86/crypto/glue_helper.ko
kernel/arch/x86/crypto/salsa20-x86_64.ko:
kernel/arch/x86/crypto/serpent-sse2-x86_64.ko: kernel/crypto/xts.ko kernel/crypto/serpent_generic.ko kernel/crypto/lrw.ko kernel/crypto/gf128mul.ko kernel/arch/x86/crypto/glue_helper.ko kernel/crypto/ablk_helper.ko kernel/crypto/cryptd.ko
kernel/arch/x86/crypto/aesni-intel.ko: kernel/arch/x86/crypto/aes-x86_64.ko kernel/crypto/lrw.ko kernel/crypto/gf128mul.ko kernel/arch/x86/crypto/glue_helper.ko kernel/crypto/ablk_helper.ko kernel/crypto/cryptd.ko
kernel/arch/x86/crypto/ghash-clmulni-intel.ko: kernel/crypto/cryptd.ko
kernel/arch/x86/crypto/sha1-ssse3.ko:
... 此处省略几千行
```

* `modprobe --show-dependency xxx.ko`，以 lsmod 不同的是，该命令显示谁依赖 xxx.ko。

```
sunyongfeng@ubuntu:~$ modprobe --show-depends -n iptable_filter
insmod /lib/modules/3.19.8-031908-generic/kernel/net/netfilter/x_tables.ko 
insmod /lib/modules/3.19.8-031908-generic/kernel/net/ipv4/netfilter/ip_tables.ko 
insmod /lib/modules/3.19.8-031908-generic/kernel/net/ipv4/netfilter/iptable_filter.ko 
```

* depmod，生成 modules.dep 和 map 文件

## 依赖相关的问题
### 提示 "modprobe: ERROR: ../libkmod/libkmod.c:5"

可能出现的原因:

* 内核版本不一致
* modules.dep.bin 受损

参考:

* [“Could not open moddep file '/lib/modules/3.XX-generic/modules.dep.bin'” when mounting using a loop](https://askubuntu.com/questions/459296/could-not-open-moddep-file-lib-modules-3-xx-generic-modules-dep-bin-when-mo) 
* [使用循环进行挂载时的modprobe"无法打开moddep文件'/lib/modules/3.XX generic/modules.dep.bin'"](https://www.helplib.com/ubuntu/article_159091)

解决:（思路，重新生成 modules.dep.bin，或干脆重新替换内核）

* `depmod`
* uname -r, `apt-get install --reinstall linux-image-'uname -r 的结果'`

一个样例，发现 docker 服务起不来，通过 systemd 的 log 查看（`sudo journalctl -u systemd-modules-load.service -b`）是 modprobe 其他模块失败，导致 docker 服务退出。

```
Nov 09 11:07:18 hostnamexxx systemd[1]: Starting Docker Application Container Engine...
Nov 09 11:07:18 hostnamexxx docker[1019]: time="2018-11-09T11:07:18.725826063+08:00" level=info msg="New containerd process, pid: 1024\n"
Nov 09 11:07:19 hostnamexxx docker[1019]: time="2018-11-09T11:07:19.800079405+08:00" level=info msg="Graph migration to content-addressability took 0.00 seconds"
Nov 09 11:07:19 hostnamexxx docker[1019]: time="2018-11-09T11:07:19.801535791+08:00" level=warning msg="Running modprobe bridge br_netfilter failed with message: modprobe: ERROR: ../libkmod/libkmod.c:557 kmod_search_moddep() could not open moddep file '/lib/modules/3.16.0-5-amd64/modules.dep.bin'\nmodprobe: ERROR: ../libkmod/libkmod.c:557 kmod_search_moddep() could not open moddep file '/lib/modules/3.16.0-5-amd64/modules.dep.bin'\n, error: exit status 1"
Nov 09 11:07:19 hostnamexxx docker[1019]: time="2018-11-09T11:07:19.802736912+08:00" level=warning msg="Running modprobe nf_nat failed with message: `modprobe: ERROR: ../libkmod/libkmod.c:557 kmod_search_moddep() could not open moddep file '/lib/modules/3.16.0-5-amd64/modules.dep.bin'`, error: exit status 1"
Nov 09 11:07:19 hostnamexxx docker[1019]: time="2018-11-09T11:07:19.803828391+08:00" level=warning msg="Running modprobe xt_conntrack failed with message: `modprobe: ERROR: ../libkmod/libkmod.c:557 kmod_search_moddep() could not open moddep file '/lib/modules/3.16.0-5-amd64/modules.dep.bin'`, error: exit status 1"
Nov 09 11:07:19 hostnamexxx docker[1019]: time="2018-11-09T11:07:19.805611702+08:00" level=info msg="Firewalld running: false"
Nov 09 11:07:19 hostnamexxx docker[1019]: time="2018-11-09T11:07:19.904012372+08:00" level=fatal msg="Error starting daemon: Error initializing network controller: Error creating default \"bridge\" network: package not installed"
Nov 09 11:07:19 hostnamexxx systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
Nov 09 11:07:19 hostnamexxx docker[1019]: time="2018-11-09T11:07:19+08:00" level=info msg="stopping containerd after receiving terminated"
Nov 09 11:07:20 hostnamexxx systemd[1]: Failed to start Docker Application Container Engine.
Nov 09 11:07:20 hostnamexxx systemd[1]: Unit docker.service entered failed state.
```

运行一下 `sudo depmod`，pass。
