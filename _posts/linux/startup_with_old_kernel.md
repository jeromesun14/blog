title: ubuntu 用回旧版内核
date: 2017-04-18 15:46:20
toc: true
tags: [Linux, ubuntu, grub]
categories: linux
keywords: [ubuntu, grub, kernel, version, out of date, old, 16.04, 3.19, 4.4.0]
description: ubuntu 下载旧版内核，并修改 grub，默认启动旧版内核。
---

尝试跑 p4factory app int，其 vxlan 内核模块目前只支持内核版本 3.19，而当前 ubuntu 16.04 默认内核版本都为 16.04，内核模块编译不过，因此需要安装旧版内核并默认启动旧版内核。

完整流程
-------
### 下载旧 Linux 内核版本

见 [askubuntu](https://askubuntu.com/questions/598483/how-can-i-use-kernel-3-19-in-14-04-now/626860) David Foerster 的答复。从 [Kernel/MainlineBuilds](https://wiki.ubuntu.com/Kernel/MainlineBuilds?action=show&redirect=KernelMainlineBuilds#Installing_upstream_kernels) 下载旧版内核 deb，[链接](http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.19.8-vivid/)，这里以 3.19.8-vivid 为例。

* headers, [32-bit, 64-bit](http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.19.8-vivid/linux-headers-3.19.8-031908_3.19.8-031908.201505110938_all.deb)
* images, [32-bit](http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.19.8-vivid/linux-image-3.19.8-031908-generic_3.19.8-031908.201505110938_i386.deb), [64-bit](http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.19.8-vivid/linux-image-3.19.8-031908-generic_3.19.8-031908.201505110938_amd64.deb)

### 安装旧 Linux 内核版本

1. 安装依赖`sudo apt-get install module-init-tools`
2. 修改 `/usr/src/linux-headers-3.19.8-031908-generic/include/generated/utsrelease.h` 宏 `#define UTS_UBUNTU_RELEASE_ABI 031908` 为 `#define UTS_UBUNTU_RELEASE_ABI 31908`。
3. 使用 `sudo dpkg -i xxx.deb`，将 xxx 替换为你下载的 headers 和 images。

#### 失败log - 未安装 module-init-tools

```
sunyongfeng@openswitch-OptiPlex-380:~/Downloads$ sudo dpkg -i linux-headers-3.19.8-031908_3.19.8-031908.201505110938_all.deb 
Selecting previously unselected package linux-headers-3.19.8-031908.
(Reading database ... 366654 files and directories currently installed.)
Preparing to unpack linux-headers-3.19.8-031908_3.19.8-031908.201505110938_all.deb ...
Unpacking linux-headers-3.19.8-031908 (3.19.8-031908.201505110938) ...
Setting up linux-headers-3.19.8-031908 (3.19.8-031908.201505110938) ...
sunyongfeng@openswitch-OptiPlex-380:~/Downloads$ sudo dpkg -i linux-headers-3.19.8-031908
linux-headers-3.19.8-031908_3.19.8-031908.201505110938_all.deb            linux-headers-3.19.8-031908-generic_3.19.8-031908.201505110938_amd64.deb
sunyongfeng@openswitch-OptiPlex-380:~/Downloads$ sudo dpkg -i linux-image-3.19.8-031908-generic_3.19.8-031908.201505110938_amd64.deb 
Selecting previously unselected package linux-image-3.19.8-031908-generic.
(Reading database ... 382437 files and directories currently installed.)
Preparing to unpack linux-image-3.19.8-031908-generic_3.19.8-031908.201505110938_amd64.deb ...
Done.
Unpacking linux-image-3.19.8-031908-generic (3.19.8-031908.201505110938) ...
dpkg: dependency problems prevent configuration of linux-image-3.19.8-031908-generic:
 linux-image-3.19.8-031908-generic depends on module-init-tools (>= 3.3-pre11-4ubuntu3); however:
  Package module-init-tools is not installed.

dpkg: error processing package linux-image-3.19.8-031908-generic (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 linux-image-3.19.8-031908-generic
```

#### 失败 log - 未修改 utsrelease.h

```
sunyongfeng@openswitch-OptiPlex-380:~/Downloads$ sudo apt-get install module-init-tools 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-headers-4.4.0-66 linux-headers-4.4.0-66-generic linux-headers-4.4.0-70 linux-headers-4.4.0-70-generic linux-image-4.4.0-66-generic
  linux-image-4.4.0-70-generic linux-image-extra-4.4.0-66-generic linux-image-extra-4.4.0-70-generic linux-tools-4.4.0-66 linux-tools-4.4.0-66-generic
  linux-tools-4.4.0-70 linux-tools-4.4.0-70-generic
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  module-init-tools
0 upgraded, 1 newly installed, 0 to remove and 261 not upgraded.
2 not fully installed or removed.
Need to get 2,320 B of archives.
After this operation, 18.4 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu xenial/universe amd64 module-init-tools all 22-1ubuntu4 [2,320 B]
Fetched 2,320 B in 5s (449 B/s)               
Selecting previously unselected package module-init-tools.
(Reading database ... 387755 files and directories currently installed.)
Preparing to unpack .../module-init-tools_22-1ubuntu4_all.deb ...
Unpacking module-init-tools (22-1ubuntu4) ...
Setting up module-init-tools (22-1ubuntu4) ...
Setting up linux-image-3.19.8-031908-generic (3.19.8-031908.201505110938) ...
Running depmod.
update-initramfs: deferring update (hook will be called later)
Examining /etc/kernel/postinst.d.
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/dkms 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
ERROR (dkms apport): kernel package linux-headers-3.19.8-031908-generic is not supported
Error! Bad return status for module build on kernel: 3.19.8-031908-generic (x86_64)
Consult /var/lib/dkms/lttng-modules/2.8.0/build/make.log for more information.
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
update-initramfs: Generating /boot/initrd.img-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/pm-utils 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/unattended-upgrades 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/update-notifier 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/zz-update-grub 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-4.4.0-72-generic
Found initrd image: /boot/initrd.img-4.4.0-72-generic
Found linux image: /boot/vmlinuz-4.4.0-71-generic
Found initrd image: /boot/initrd.img-4.4.0-71-generic
Found linux image: /boot/vmlinuz-4.4.0-70-generic
Found initrd image: /boot/initrd.img-4.4.0-70-generic
Found linux image: /boot/vmlinuz-4.4.0-66-generic
Found initrd image: /boot/initrd.img-4.4.0-66-generic
Found linux image: /boot/vmlinuz-4.4.0-64-generic
Found initrd image: /boot/initrd.img-4.4.0-64-generic
Found linux image: /boot/vmlinuz-4.4.0-45-generic
Found initrd image: /boot/initrd.img-4.4.0-45-generic
Found linux image: /boot/vmlinuz-3.19.8-031908-generic
Found initrd image: /boot/initrd.img-3.19.8-031908-generic
Found memtest86+ image: /boot/memtest86+.elf
Found memtest86+ image: /boot/memtest86+.bin
done
Setting up linux-headers-3.19.8-031908-generic (3.19.8-031908.201505110938) ...
Examining /etc/kernel/header_postinst.d.
run-parts: executing /etc/kernel/header_postinst.d/dkms 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
ERROR (dkms apport): kernel package linux-headers-3.19.8-031908-generic is not supported
Error! Bad return status for module build on kernel: 3.19.8-031908-generic (x86_64)
Consult /var/lib/dkms/lttng-modules/2.8.0/build/make.log for more information.
```

查看 make.log 发现，dkms 将 `/usr/src/linux-headers-3.19.8-031908-generic/include/generated/utsrelease.h` 的十进制数宏 `#define UTS_UBUNTU_RELEASE_ABI 031908` 用成八进制数（估计是不兼容旧版本导致，不过 utsrelease.h 的这种写法也很别扭，因为默认使用 `0` 做为前导码标识八进制数）。将该宏改为 `#define UTS_UBUNTU_RELEASE_ABI 31908` 即可。

```
sunyongfeng@openswitch-OptiPlex-380:~/Downloads$ vi /var/lib/dkms/lttng-modules/2.8.0/build/make.log
  1 DKMS make.log for lttng-modules-2.8.0 for kernel 3.19.8-031908-generic (x86_64)                     
  2 2017年 04月 18日 星期二 14:50:53 CST                                                                
  3 make: Entering directory '/usr/src/linux-headers-3.19.8-031908-generic'                             
  4   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-ring-buffer-client-discard.o                
  5   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-ring-buffer-client-overwrite.o              
  6   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-ring-buffer-metadata-client.o               
  7   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-ring-buffer-client-mmap-discard.o           
  8   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-ring-buffer-client-mmap-overwrite.o         
  9   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-ring-buffer-metadata-mmap-client.o          
 10   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-clock.o                                     
 11   CC [M]  /var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.o                                    
 12 In file included from /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:29:0,          
 13                  from /var/lib/dkms/lttng-modules/2.8.0/build/wrapper/page_alloc.h:28,              
 14                  from /var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.c:27:                    
 15 include/generated/utsrelease.h:2:32: error: invalid digit "9" in octal constant                     
 16  #define UTS_UBUNTU_RELEASE_ABI 031908                                                              
 17                                 ^                                                                   
 18 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:49:31: note: in expansion of macro ‘UTS_UBUNTU_RELEASE_ABI’
 19   ((LINUX_VERSION_CODE << 8) + UTS_UBUNTU_RELEASE_ABI)                                              
 20                                ^                                                                    
 21 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:56:3: note: in expansion of macro ‘LTTNG_UBUNTU_VERSION_CODE’
 22   (LTTNG_UBUNTU_VERSION_CODE >= \                                                                   
 23    ^                                                                                                
 24 /var/lib/dkms/lttng-modules/2.8.0/build/wrapper/page_alloc.h:39:6: note: in expansion of macro ‘LTTNG_UBUNTU_KERNEL_RANGE’
 25    || LTTNG_UBUNTU_KERNEL_RANGE(3,16,7,34, 3,17,0,0)))                                              
 26       ^                                                                                             
 27 include/generated/utsrelease.h:2:32: error: invalid digit "9" in octal constant                     
 28  #define UTS_UBUNTU_RELEASE_ABI 031908                                                              
 29                                 ^                                                                   
 30 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:49:31: note: in expansion of macro ‘UTS_UBUNTU_RELEASE_ABI’
 31   ((LINUX_VERSION_CODE << 8) + UTS_UBUNTU_RELEASE_ABI)                                              
 32                                ^                                                                    
 33 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:58:3: note: in expansion of macro ‘LTTNG_UBUNTU_VERSION_CODE’
 34    LTTNG_UBUNTU_VERSION_CODE < \                                                                    
 35    ^                                                                                                
 36 /var/lib/dkms/lttng-modules/2.8.0/build/wrapper/page_alloc.h:39:6: note: in expansion of macro ‘LTTNG_UBUNTU_KERNEL_RANGE’
 37    || LTTNG_UBUNTU_KERNEL_RANGE(3,16,7,34, 3,17,0,0)))                                              
 38       ^                                                                                             
 39 include/generated/utsrelease.h:2:32: error: invalid digit "9" in octal constant                     
 40  #define UTS_UBUNTU_RELEASE_ABI 031908                                                              
 41                                 ^                                                                   
 42 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:49:31: note: in expansion of macro ‘UTS_UBUNTU_RELEASE_ABI’
 43   ((LINUX_VERSION_CODE << 8) + UTS_UBUNTU_RELEASE_ABI)                                              
 44                                ^                                                                    
 45 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:56:3: note: in expansion of macro ‘LTTNG_UBUNTU_VERSION_CODE’
 46   (LTTNG_UBUNTU_VERSION_CODE >= \                                                                   
 47    ^                                                                                                
 48 /var/lib/dkms/lttng-modules/2.8.0/build/wrapper/page_alloc.h:71:5: note: in expansion of macro ‘LTTNG_UBUNTU_KERNEL_RANGE’
 49   && LTTNG_UBUNTU_KERNEL_RANGE(3,13,11,50, 3,14,0,0))                                               
 50      ^                                                                                              
 51 include/generated/utsrelease.h:2:32: error: invalid digit "9" in octal constant                     
 52  #define UTS_UBUNTU_RELEASE_ABI 031908                                                              
 53                                 ^                                                                   
 54 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:49:31: note: in expansion of macro ‘UTS_UBUNTU_RELEASE_ABI’
 55   ((LINUX_VERSION_CODE << 8) + UTS_UBUNTU_RELEASE_ABI)                                              
 56                                ^                                                                    
 57 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:58:3: note: in expansion of macro ‘LTTNG_UBUNTU_VERSION_CODE’
 58    LTTNG_UBUNTU_VERSION_CODE < \                                                                    
 59    ^                                                                                                
 60 /var/lib/dkms/lttng-modules/2.8.0/build/wrapper/page_alloc.h:71:5: note: in expansion of macro ‘LTTNG_UBUNTU_KERNEL_RANGE’
 61   && LTTNG_UBUNTU_KERNEL_RANGE(3,13,11,50, 3,14,0,0))                                               
 62      ^                                                                                              
 63 scripts/Makefile.build:257: recipe for target '/var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.o' failed
 64 make[1]: *** [/var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.o] Error 1                       
 65 Makefile:1382: recipe for target '_module_/var/lib/dkms/lttng-modules/2.8.0/build' failed           
 66 make: *** [_module_/var/lib/dkms/lttng-modules/2.8.0/build] Error 2                                 
 57 /var/lib/dkms/lttng-modules/2.8.0/build/lttng-kernel-version.h:58:3: note: in expansion of macro ‘LTTNG_UBUNTU_VERSION_CODE’
 58    LTTNG_UBUNTU_VERSION_CODE < \                                                                    
 59    ^                                                                                                
 60 /var/lib/dkms/lttng-modules/2.8.0/build/wrapper/page_alloc.h:71:5: note: in expansion of macro ‘LTTNG_UBUNTU_KERNEL_RANGE’
 61   && LTTNG_UBUNTU_KERNEL_RANGE(3,13,11,50, 3,14,0,0))                                               
 62      ^                                                                                              
 63 scripts/Makefile.build:257: recipe for target '/var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.o' failed
 64 make[1]: *** [/var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.o] Error 1                       
 65 Makefile:1382: recipe for target '_module_/var/lib/dkms/lttng-modules/2.8.0/build' failed           
 66 make: *** [_module_/var/lib/dkms/lttng-modules/2.8.0/build] Error 2                                 
 58    LTTNG_UBUNTU_VERSION_CODE < \                                                                    
 59    ^                                                                                                
 60 /var/lib/dkms/lttng-modules/2.8.0/build/wrapper/page_alloc.h:71:5: note: in expansion of macro ‘LTTNG_UBUNTU_KERNEL_RANGE’
 61   && LTTNG_UBUNTU_KERNEL_RANGE(3,13,11,50, 3,14,0,0))                                               
 62      ^                                                                                              
 63 scripts/Makefile.build:257: recipe for target '/var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.o' failed
 64 make[1]: *** [/var/lib/dkms/lttng-modules/2.8.0/build/lttng-events.o] Error 1                       
 65 Makefile:1382: recipe for target '_module_/var/lib/dkms/lttng-modules/2.8.0/build' failed           
 66 make: *** [_module_/var/lib/dkms/lttng-modules/2.8.0/build] Error 2                                 
 67 make: Leaving directory '/usr/src/linux-headers-3.19.8-031908-generic'                              
  ⮀ NORMAL ⮀ ⭤ /var/lib/dkms/lttng-modules/2.8.0/build/make.log   ⮀                                                         ⮂ unix ⮃ utf-8 ⮃ no ft ⮂ 100% ⮂ ⭡  67:27 
```

修改后 log：

```
sunyongfeng@openswitch-OptiPlex-380:~/Downloads$ sudo dpkg -i linux-image-3.19.8-031908-generic_3.19.8-031908.201505110938_amd64.deb 
(Reading database ... 387757 files and directories currently installed.)
Preparing to unpack linux-image-3.19.8-031908-generic_3.19.8-031908.201505110938_amd64.deb ...
Done.
Unpacking linux-image-3.19.8-031908-generic (3.19.8-031908.201505110938) over (3.19.8-031908.201505110938) ...
Examining /etc/kernel/postrm.d .
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
Setting up linux-image-3.19.8-031908-generic (3.19.8-031908.201505110938) ...
Running depmod.
update-initramfs: deferring update (hook will be called later)
Not updating initrd symbolic links since we are being updated/reinstalled 
(3.19.8-031908.201505110938 was configured last, according to dpkg)
Not updating image symbolic links since we are being updated/reinstalled 
(3.19.8-031908.201505110938 was configured last, according to dpkg)
Examining /etc/kernel/postinst.d.
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/dkms 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
update-initramfs: Generating /boot/initrd.img-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/pm-utils 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/unattended-upgrades 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/update-notifier 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
run-parts: executing /etc/kernel/postinst.d/zz-update-grub 3.19.8-031908-generic /boot/vmlinuz-3.19.8-031908-generic
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-4.4.0-72-generic
Found initrd image: /boot/initrd.img-4.4.0-72-generic
Found linux image: /boot/vmlinuz-4.4.0-71-generic
Found initrd image: /boot/initrd.img-4.4.0-71-generic
Found linux image: /boot/vmlinuz-4.4.0-70-generic
Found initrd image: /boot/initrd.img-4.4.0-70-generic
Found linux image: /boot/vmlinuz-4.4.0-66-generic
Found initrd image: /boot/initrd.img-4.4.0-66-generic
Found linux image: /boot/vmlinuz-4.4.0-64-generic
Found initrd image: /boot/initrd.img-4.4.0-64-generic
Found linux image: /boot/vmlinuz-4.4.0-45-generic
Found initrd image: /boot/initrd.img-4.4.0-45-generic
Found linux image: /boot/vmlinuz-3.19.8-031908-generic
Found initrd image: /boot/initrd.img-3.19.8-031908-generic
Found memtest86+ image: /boot/memtest86+.elf
Found memtest86+ image: /boot/memtest86+.bin
done
sunyongfeng@openswitch-OptiPlex-380:~/Downloads$ 
```

### 修改 grub

详见 [Set “older” kernel as default grub entry](https://askubuntu.com/questions/216398/set-older-kernel-as-default-grub-entry) 中 DaimyoKirby 的回答。

* 备份 grub 配置，`sudo cp /etc/default/grub /etc/default/grub.bak`
* 修改 grub 配置，`sudo vi /etc/default/grub`，将 `GRUB_DEFAULT=0` 改为 `GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 3.19.8-031908-generic"`。
  + `Advanced options for Ubuntu` 表示 grub 的第一级菜单
  + `>` 表示接下一级菜单
  + `Ubuntu, with Linux 3.19.8-031908-generic` 表示第二级菜单
  + 以上菜单都可在 `/boot/grub/grub.cfg` 中指到原始字符
* 更新 grub，`sudo update-grub`

### 确认修订成功

更新 grub 后，`sudo shutdown -r 0` 重启设备，重新登陆设备，通过 `uname -a` 确认内核已更新为旧版内核。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory$ uname -a
Linux openswitch-OptiPlex-380 3.19.8-031908-generic #201505110938 SMP Mon May 11 13:39:59 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```
