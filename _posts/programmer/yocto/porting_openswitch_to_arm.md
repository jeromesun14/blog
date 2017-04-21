title: 移植 openswitch 到 arm
date: 2017-04-21 11:02:20
toc: true
tags: [Linux, yocto, openswitch]
categories: yocto
keywords: [yocto, bitbake, openswitch, openembedded, oe, ops, download too slowly, 下载太慢, sstate, gcc, tune, compile, long, 编译太久]
description: 记录移值 HP openswitch 到 arm CPU 的过程。
------------

HP ops 目前已过时，现在主导 openswitch 项目的是 DELL 和 SnapRoute，主推 DELL NAS + SnapRoute Flexswitch （称为 opx）。

本文 openswitch 指 HP ops。记录以前移植 openswtich 到 arm CPU 的过程。

1. BSP 适配
-------------
### 1.1. 新增一个新产品
在目录 `yocto/openswitch/` 下新建 `meta-platform-openswitch-[product]`，其中 product 对应新增的产品名字，可以从已有的产品目录拷贝。以 xxx 为例子：
* 使用 cp 命令，拷贝其他产品目录为 `meta-platform-openswitch-xxx`：
```
cp meta-platform-openswitch-as6712  meta-platform-openswitch-xxx –r
```
* 删除 `recipes-kernel` 以外的配方目录（以 `recipes-*` 开头的）
* 将目录下所有文件的中 as6712 修改为 xxx，主要有以下几个文件：
    + `conf/machine/as6712.conf` 重命名为 `conf/machine/xxx.conf`
    + `conf/layer/conf`
    + `README`
    + `rules.make`
    + `recipes-kernel` 子目录下的 `*.bbappend` 中的 `PR_append`
* 根据 cpu 架构适配 `conf/machine/xxx.conf` 文件，xxx 的 cpu 是 arm cortexa9 系列，其中 `DEFAULTTUNE` 定义表示不支持硬件 FPU。

```
DEFAULTTUNE ?= "core2-64"
require conf/machine/include/tune-core2.inc
改成：
DEFAULTTUNE ?= "cortexa9-neon"
require conf/machine/include/tune-cortexa9.inc
```

* 根据产品支持的内核版本，修订 `PREFERRED_VERSION_linux-ops` 参数
* 暂时屏蔽 `MACHINE_FEATURES` 和 `MACHINE_ESSENTIAL_EXTRA_RDEPENDS`，根据需要添加          
 
### 1.2.  添加一个新内核版本支持
* 在目录 `yocto/openswitch/meta-distro-openswitch/recipes-kernel/linux/` 新增 `linux-ops_[version].bb` 文件，可以从其他版本复制
* 修订 *.bb 文件
    + 对应内核版本号，`KERNEL_RELEASE = "3.10.18"`
    + 对应内核源码下载地址，当前为本地地址 `SRC_URI = http://192.168.x.x/linux-3.10.18.tar.gz;name=kernel`
    + 对应源码的 md5sum 值（可以使用 linux md5sum 工具计算），`SRC_URI[kernel.md5sum] = "e19cbb242d776223c337e10a31ca9865"`
    + 对应源码的 sha256sum 值（可以使用 linux sha256sum 工具计算），`SRC_URI[kernel.sha256sum]     = "a11abd0ac80c6195aaab16c8be3e97c07c705d0ee135f3c5c092a99c4973b1be"`
 
> 备注：两个 checksum 和下载地址要保证正确，不然编译的时候会出错
 
* 由于 openswitch 的 linux-ops 不支持设备树，需要新增文件 `linux-dtb.inc`，并且修订文件 `linux.inc`：

```
LOCALVERSION ?= ""
require linux-dtb.inc     #新增
#kernel_conf_variable CMDLINE "\"${CMDLINE} ${CMDLINE_DEBUG}\""
```

* 适配 xxx，新增文件：

```
yocto/openswitch/meta-platform-openswitch-xxx/recipes-kernel/dtc/dtc_git.bbappend
yocto/openswitch/meta-platform-openswitch-xxx/recipes-kernel/linux/linux-ops_3.10.18.bbappend
```

* 在 `meta-platform-openswitch-xxx/recipes-kernel/linux/linux-ops/` 增加内核配置文件以及产品需要的 patch文件
 
### 1.3. 适配完成之后，使用 `make kernel` 编译内核模块
no log。
 
### 1.4. 调试问题汇总
* Open-switch 内核默认编译出来的 image 是 zimage，我的设备 uboot 比较老，不支持这种格式；可以通过在 `xxx.config` 中新增 `KERNEL_IMAGETYPE = "uImage"` 来指定成 uimage 格式。
* 在设备树编译的时候，需要指定目录的在内核的绝对地址，不能单单执行个xx.dts: `KERNEL_DEVICETREE = "${S}/arch/arm/boot/dts/zzz_yyy.dts"`
* 内核编译出来的 image 如果使用 uImage 或者 zImage 的时候，内核需要打开以下配置，并且 cpu 产品目录需要有这个文件 `arch/arm/mach-yyy/include/mach/uncompress.h`，否则会出现调整内核之后死机：

```
CONFIG_BLK_DEV_INITRD=y
CONFIG_INITRAMFS_SOURCE=""
CONFIG_RD_GZIP=y
CONFIG_DECOMPRESS_GZIP=y
```

* 如果使用的 uImage 或者 zImage 格式加载，内核的入口函数是 `arch/arm/boot/bootp/init.S` 中，这个文件主要是跳转到 `misc.c` 自解压内核 image，然后跳转到真正的内核执行。
* 如果出现自解压完成，调整到真正内核执行的时候无 log 输出，可以将内核的 `early debug` 打开，具体配置宏如下：

```
CONFIG_DEBUG_LL=y
CONFIG_DEBUG_LL_UART_NONE=y
# CONFIG_DEBUG_ICEDCC is not set
# CONFIG_DEBUG_SEMIHOSTING is not set
CONFIG_DEBUG_LL_INCLUDE="mach/debug-macro.S"
CONFIG_UNCOMPRESS_INCLUDE="mach/uncompress.h"
CONFIG_EARLY_PRINTK=y
```

* 如果出现以下的错误 log，是由于 uboot 未传入正确的设备树或者参数导致：

```
10-21-09:30:03.036:Uncompressing Linux... done, booting the kernel.
10-21-09:30:03.036:
10-21-09:30:03.036:Error: unrecognized/unsupported machine ID (r1 = 0x00000bb8).
10-21-09:30:03.036:
10-21-09:30:03.036:Available machine support:
10-21-09:30:03.036:
10-21-09:30:03.051:ID (hex)        NAME
10-21-09:30:03.051:ffffffff        XXX YYY ZZZ
10-21-09:30:03.051:
```

* 使用 `bootm` 命令可以带两地址，第一个表示 image 的地址，第二个表示设备树地址，`bootm 62000000 - 62400000`。

### 1.5. kernel defconfig

### 1.6. 支持 systemd 的 kernel defconfig
这里的 defconfig 不全是支持 systemd 所需配置。
支持 systemd 所需配置见 [systemd README](https://github.com/systemd/systemd/blob/master/README)，注意不同的 systemd 版本可能对内核的配置要求不一样。

```
diff --git a/yocto/openswitch/meta-platform-openswitch-xxx/recipes-kernel/linux/linux-ops/defconfig b/yocto/openswitch/meta-platform-openswitch-xxx/recipes-kernel/linux/linux-ops/defconfig
index a61c606..06a5f0e 100755
--- a/yocto/openswitch/meta-platform-openswitch-xxx/recipes-kernel/linux/linux-ops/defconfig
+++ b/yocto/openswitch/meta-platform-openswitch-xxx/recipes-kernel/linux/linux-ops/defconfig
@@ -41,7 +41,7 @@ CONFIG_SYSVIPC=y
 CONFIG_SYSVIPC_SYSCTL=y
 CONFIG_POSIX_MQUEUE=y
 CONFIG_POSIX_MQUEUE_SYSCTL=y
-# CONFIG_FHANDLE is not set
+CONFIG_FHANDLE=y
 # CONFIG_AUDIT is not set
 CONFIG_HAVE_GENERIC_HARDIRQS=y
 
@@ -91,7 +91,15 @@ CONFIG_RCU_FANOUT_LEAF=16
 CONFIG_IKCONFIG=y
 CONFIG_IKCONFIG_PROC=y
 CONFIG_LOG_BUF_SHIFT=21
-# CONFIG_CGROUPS is not set
+CONFIG_CGROUPS=y
+# CONFIG_CGROUP_DEBUG is not set
+# CONFIG_CGROUP_FREEZER is not set
+# CONFIG_CGROUP_DEVICE is not set
+# CONFIG_CPUSETS is not set
+# CONFIG_CGROUP_CPUACCT is not set
+# CONFIG_RESOURCE_COUNTERS is not set
+# CONFIG_CGROUP_SCHED is not set
+# CONFIG_BLK_CGROUP is not set
 # CONFIG_CHECKPOINT_RESTORE is not set
 # CONFIG_NAMESPACES is not set
 CONFIG_UIDGID_CONVERTED=y
@@ -99,13 +107,7 @@ CONFIG_UIDGID_CONVERTED=y
 # CONFIG_SCHED_AUTOGROUP is not set
 # CONFIG_SYSFS_DEPRECATED is not set
 # CONFIG_RELAY is not set
-CONFIG_BLK_DEV_INITRD=y
-CONFIG_INITRAMFS_SOURCE=""
-CONFIG_RD_GZIP=y
-# CONFIG_RD_BZIP2 is not set
-# CONFIG_RD_LZMA is not set
-# CONFIG_RD_XZ is not set
-# CONFIG_RD_LZO is not set
+# CONFIG_BLK_DEV_INITRD is not set
 # CONFIG_CC_OPTIMIZE_FOR_SIZE is not set
 CONFIG_SYSCTL=y
 CONFIG_ANON_INODES=y
@@ -380,12 +382,14 @@ CONFIG_ARM_AMBA=y
 CONFIG_PCI=y
 CONFIG_PCI_DOMAINS=y
 CONFIG_PCI_SYSCALL=y
+# CONFIG_ARCH_SUPPORTS_MSI is not set
 # CONFIG_PCI_DEBUG is not set
 # CONFIG_PCI_REALLOC_ENABLE_AUTO is not set
 # CONFIG_PCI_STUB is not set
 # CONFIG_PCI_IOV is not set
 # CONFIG_PCI_PRI is not set
 # CONFIG_PCI_PASID is not set
+# CONFIG_PCIEPORTBUS is not set
 # CONFIG_PCCARD is not set
 
 #
@@ -452,10 +456,7 @@ CONFIG_ATAGS=y
 # CONFIG_DEPRECATED_PARAM_STRUCT is not set
 CONFIG_ZBOOT_ROM_TEXT=0x0
 CONFIG_ZBOOT_ROM_BSS=0x0
-CONFIG_ARM_APPENDED_DTB=y
-CONFIG_ARM_ATAG_DTB_COMPAT=y
-CONFIG_ARM_ATAG_DTB_COMPAT_CMDLINE_FROM_BOOTLOADER=y
-# CONFIG_ARM_ATAG_DTB_COMPAT_CMDLINE_EXTEND is not set
+# CONFIG_ARM_APPENDED_DTB is not set
 CONFIG_CMDLINE="console=ttyS0,115200n8 maxcpus=1 mem=240M"
 CONFIG_CMDLINE_FROM_BOOTLOADER=y
 # CONFIG_CMDLINE_EXTEND is not set
@@ -463,7 +464,7 @@ CONFIG_CMDLINE_FROM_BOOTLOADER=y
 # CONFIG_XIP_KERNEL is not set
 # CONFIG_KEXEC is not set
 # CONFIG_CRASH_DUMP is not set
-CONFIG_AUTO_ZRELADDR=y
+# CONFIG_AUTO_ZRELADDR is not set
 
 #
 # CPU Power Management
@@ -478,7 +479,7 @@ CONFIG_AUTO_ZRELADDR=y
 #
 # At least one emulation must be selected
 #
-# CONFIG_VFP is not set
+#CONFIG_VFP=y
 
 #
 # Userspace binary formats
@@ -544,6 +545,7 @@ CONFIG_HAVE_NET_DSA=y
 CONFIG_RPS=y
 CONFIG_RFS_ACCEL=y
 CONFIG_XPS=y
+# CONFIG_NETPRIO_CGROUP is not set
 CONFIG_BQL=y
 # CONFIG_BPF_JIT is not set
 
@@ -571,7 +573,8 @@ CONFIG_HAVE_BPF_JIT=y
 # Generic Driver Options
 #
 CONFIG_UEVENT_HELPER_PATH="/sbin/mdev"
-# CONFIG_DEVTMPFS is not set
+CONFIG_DEVTMPFS=y
+CONFIG_DEVTMPFS_MOUNT=y
 CONFIG_STANDALONE=y
 CONFIG_PREVENT_FIRMWARE_BUILD=y
 CONFIG_FW_LOADER=y
@@ -1760,8 +1763,15 @@ CONFIG_ARM_GIC=y
 #
 CONFIG_DCACHE_WORD_ACCESS=y
 # CONFIG_EXT2_FS is not set
-# CONFIG_EXT3_FS is not set
+CONFIG_EXT3_FS=y
+CONFIG_EXT3_DEFAULTS_TO_ORDERED=y
+CONFIG_EXT3_FS_XATTR=y
+# CONFIG_EXT3_FS_POSIX_ACL is not set
+# CONFIG_EXT3_FS_SECURITY is not set
 # CONFIG_EXT4_FS is not set
+CONFIG_JBD=y
+# CONFIG_JBD_DEBUG is not set
+CONFIG_FS_MBCACHE=y
 # CONFIG_REISERFS_FS is not set
 # CONFIG_JFS_FS is not set
 # CONFIG_XFS_FS is not set
@@ -1769,6 +1779,7 @@ CONFIG_DCACHE_WORD_ACCESS=y
 # CONFIG_BTRFS_FS is not set
 # CONFIG_NILFS2_FS is not set
 # CONFIG_FS_POSIX_ACL is not set
+CONFIG_EXPORTFS=y
 CONFIG_FILE_LOCKING=y
 CONFIG_FSNOTIFY=y
 CONFIG_DNOTIFY=y
@@ -1932,7 +1943,6 @@ CONFIG_HAVE_DEBUG_KMEMLEAK=y
 # CONFIG_DEBUG_LOCKING_API_SELFTESTS is not set
 # CONFIG_DEBUG_STACK_USAGE is not set
 # CONFIG_DEBUG_KOBJECT is not set
-# CONFIG_DEBUG_HIGHMEM is not set
 # CONFIG_DEBUG_BUGVERBOSE is not set
 # CONFIG_DEBUG_INFO is not set
 # CONFIG_DEBUG_VM is not set
@@ -1983,13 +1993,9 @@ CONFIG_HAVE_ARCH_KGDB=y
 # CONFIG_STRICT_DEVMEM is not set
 # CONFIG_ARM_UNWIND is not set
 CONFIG_DEBUG_USER=y
-CONFIG_DEBUG_LL=y
-CONFIG_DEBUG_LL_UART_NONE=y
-# CONFIG_DEBUG_ICEDCC is not set
-# CONFIG_DEBUG_SEMIHOSTING is not set
+# CONFIG_DEBUG_LL is not set
 CONFIG_DEBUG_LL_INCLUDE="mach/debug-macro.S"
 CONFIG_UNCOMPRESS_INCLUDE="mach/uncompress.h"
-CONFIG_EARLY_PRINTK=y
 # CONFIG_OC_ETM is not set
 # CONFIG_PID_IN_CONTEXTIDR is not set
 
@@ -2142,7 +2148,6 @@ CONFIG_LZO_COMPRESS=y
 CONFIG_LZO_DECOMPRESS=y
 # CONFIG_XZ_DEC is not set
 # CONFIG_XZ_DEC_BCJ is not set
-CONFIG_DECOMPRESS_GZIP=y
 CONFIG_HAS_IOMEM=y
 CONFIG_HAS_IOPORT=y
 CONFIG_HAS_DMA=y
```

2. gcc tune
---------------
### 2.1.rootfs 不可用
xxx cpu 为 arm cortex-a9 芯片。`DEFAULTTUNE` 置为 `cortexa9` 或 `cortexa9-neon` 时，编译出来的 rootfs 不可用：
启机运行时，会出现 `undefined instruction`，内核 kill init 退出：

```
init (1): undefined instruction: pc=b6ebcbdc
Code: e1866315 ee021b90 e2402020 e1866235 (f37204a1)
Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
 
CPU1: stopping
CPU: 1 PID: 0 Comm: swapper/1 Tainted: G        W    3.10.18 #1
Backtrace:
[<c00117b8>] (dump_backtrace+0x0/0x10c) from [<c00119cc>] (show_stack+0x18/0x1c)
r6:00000000 r5:00000001 r4:c041d9f8 r3:00000000
[<c00119b4>] (show_stack+0x0/0x1c) from [<c02e2034>] (dump_stack+0x24/0x28)
[<c02e2010>] (dump_stack+0x0/0x28) from [<c0013ca4>] (handle_IPI+0x118/0x134)
[<c0013b8c>] (handle_IPI+0x0/0x134) from [<c0008620>] (gic_handle_irq+0x60/0x64)
r6:ef06ff68 r5:c04022f4 r4:fee2010c r3:c000f14c
[<c00085c0>] (gic_handle_irq+0x0/0x64) from [<c000e0c0>] (__irq_svc+0x40/0x50)
Exception stack(0xef06ff68 to 0xef06ffb0)
ff60:                   c18116b0 00000000 00002fdc 00000000 ef06e000 ef06e000
ff80: c0401f28 c0401f70 c02e7b2c ef06e000 ef06e000 ef06ffbc ef06ffc0 ef06ffb0
ffa0: c000f14c c000f150 60000113 ffffffff
r7:ef06ff9c r6:ffffffff r5:60000113 r4:c000f150
[<c000f11c>] (arch_cpu_idle+0x0/0x38) from [<c0055c60>] (cpu_startup_entry+0xec/0x13c)
[<c0055b74>] (cpu_startup_entry+0x0/0x13c) from [<c02debd8>] (secondary_start_kernel+0x128/0x130)
r7:c041da1c
[<c02deab0>] (secondary_start_kernel+0x0/0x130) from [<612de104>] (0x612de104)
r4:9003c06a r3:c02de0ec
```

这个问题排查了约两个星期，排查的方向：
* 内核是否支持 systemd？
  根据 systemd 官方说明文档配置内核，启机还是挂在同样的地方；
* rootfs 的格式是不是有问题（ubifs 还是 ext3）？
  把 rootfs 改为 ubifs，启机还是挂在同样的地方；
* 反汇编
  通过 `objdump` vmlinux 或看system map table，都没有对应 PC 值。这个 undefined instruction 的意思应该就是说明没有这个指令，所以不用看了，找也找不着。

偶然的机会，与同事的讨论中，他曾经在某个产品上使用 debian 8 （默认采用 systemd） 的 rootfs 跑过。因此通过 ops-build 编译的 kernel + debian 8 rootfs （U盘启动）确认内核没有问题。
既然内核没有问题，那就得看 debian jessie arm 版本编译的结果与 ops-build 编译结果的差异，通过 `readelf -A systemd` 查看 systemd 编译结果的架构相关信息，发现了差异：

不可用的 rootfs：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/rgosm-build/prj_xxx/images/rootfs/lib/systemd$ arm-cortex_a9-linux-gnueabi-readelf -A systemd
Attribute Section: aeabi
File Attributes
  Tag_CPU_name: "7-A"
  Tag_CPU_arch: v7
  Tag_CPU_arch_profile: Application
  Tag_ARM_ISA_use: Yes
  Tag_THUMB_ISA_use: Thumb-2
  Tag_VFP_arch: VFPv3
  Tag_Advanced_SIMD_arch: NEONv1
  Tag_ABI_PCS_wchar_t: 4
  Tag_ABI_FP_rounding: Needed
  Tag_ABI_FP_denormal: Needed
  Tag_ABI_FP_exceptions: Needed
  Tag_ABI_FP_number_model: IEEE 754
  Tag_ABI_align8_needed: Yes
  Tag_ABI_enum_size: int
  Tag_ABI_HardFP_use: SP and DP
  Tag_CPU_unaligned_access: v6
```

可用的 rootfs：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/rgosm-build/prj_xxx/images/rootfs$ arm-cortex_a9-linux-gnueabi-readelf -A lib/systemd/systemd
Attribute Section: aeabi
File Attributes
  Tag_CPU_name: "4T"
  Tag_CPU_arch: v4T
  Tag_ARM_ISA_use: Yes
  Tag_THUMB_ISA_use: Thumb-1
  Tag_ABI_PCS_wchar_t: 4
  Tag_ABI_FP_rounding: Needed
  Tag_ABI_FP_denormal: Needed
  Tag_ABI_FP_exceptions: Needed
  Tag_ABI_FP_number_model: IEEE 754
  Tag_ABI_align8_needed: Yes
  Tag_ABI_align8_preserved: Yes, except leaf SP
  Tag_ABI_enum_size: int
```

主要差异在 NEON、VFP 和 HardFP 上。因尝试改 `tune-cortexa9.inc` 等相关文件调整默认 FPU 等编译选项，但是一直修改失败，因此换了个思路，看 debian porting to arm 用了什么编译选项。详见 [debian 移植到 arm 32 的编译选项](https://wiki.debian.org/ArmEabiPort) ，根据文中的建议 tune gcc。

### 2.2. tune gcc
* 配置 `yocto/openswitch/meta-platform-openswitch-xxx/conf/machine/xxx.conf`，改 `DEFAULTTUNE` 为 `armv4t`

```
DEFAULTTUNE ?= "armv4t"
require conf/machine/include/tune-cortexa9.inc
```

* 配置 `yocto/poky/meta/conf/machine/include/arm/arch-arm.inc`，改 `TARGET_FPU` 为 `soft`，添加选项 ` -mabi=aapcs-linux`
* 配置 `yocto/poky/meta/conf/machine/include/arm/feature-arm-neon.inc` 和 `yocto/poky/meta/conf/machine/include/arm/feature-arm-vfp.inc`，把 FPU 相关配置改为 soft。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ops-build.rj$ git diff yocto/poky/meta/conf/machine*
diff --git a/yocto/poky/meta/conf/machine/include/arm/arch-arm.inc b/yocto/poky/meta/conf/machine/include/arm/arch-arm.inc
index 90b80c4..b1a04b8 100644
--- a/yocto/poky/meta/conf/machine/include/arm/arch-arm.inc
+++ b/yocto/poky/meta/conf/machine/include/arm/arch-arm.inc
@@ -13,5 +13,6 @@ TUNE_PKGARCH = "${ARMPKGARCH}${ARMPKGSFX_THUMB}${ARMPKGSFX_DSP}${ARMPKGSFX_EABI}
 
ABIEXTENSION = "eabi"
 
-TARGET_FPU = "${@d.getVar('ARMPKGSFX_FPU', True).strip('-') or 'soft'}"
+TARGET_FPU = "soft"
+TUNE_CCARGS .= " -mabi=aapcs-linux"
 
diff --git a/yocto/poky/meta/conf/machine/include/arm/feature-arm-neon.inc b/yocto/poky/meta/conf/machine/include/arm/feature-arm-neon.inc
index e8b2b85..25d612e 100644
--- a/yocto/poky/meta/conf/machine/include/arm/feature-arm-neon.inc
+++ b/yocto/poky/meta/conf/machine/include/arm/feature-arm-neon.inc
@@ -1,3 +1,3 @@
TUNEVALID[neon] = "Enable Neon SIMD accelerator unit."
-TUNE_CCARGS .= "${@bb.utils.contains("TUNE_FEATURES", "neon", bb.utils.contains("TUNE_FEATURES", "vfpv4", " -mfpu=neon-vfpv4", " -mfpu=neon", d), "" , d)}"
-ARMPKGSFX_FPU .= "${@bb.utils.contains("TUNE_FEATURES", "neon", "-neon", "", d)}"
+TUNE_CCARGS .= "${@bb.utils.contains("TUNE_FEATURES", "neon", bb.utils.contains("TUNE_FEATURES", "vfpv4", " -msoft-float", " -msoft-float", d), "" , d)}"
+ARMPKGSFX_FPU .= "${@bb.utils.contains("TUNE_FEATURES", "neon", "", "", d)}"
diff --git a/yocto/poky/meta/conf/machine/include/arm/feature-arm-vfp.inc b/yocto/poky/meta/conf/machine/include/arm/feature-arm-vfp.inc
index 13927ff..73efcb8 100644
--- a/yocto/poky/meta/conf/machine/include/arm/feature-arm-vfp.inc
+++ b/yocto/poky/meta/conf/machine/include/arm/feature-arm-vfp.inc
@@ -2,8 +2,8 @@ TUNEVALID[vfp] = "Enable Vector Floating Point (vfp) unit."
ARMPKGSFX_FPU .= "${@bb.utils.contains("TUNE_FEATURES", "vfp", "-vfp", "" ,d)}"
 
TUNEVALID[vfpv4] = "Enable Vector Floating Point Version 4 (vfpv4) unit."
-ARMPKGSFX_FPU .= "${@bb.utils.contains("TUNE_FEATURES", "vfpv4", "-vfpv4", "" ,d)}"
+ARMPKGSFX_FPU .= "${@bb.utils.contains("TUNE_FEATURES", "vfpv4", "-vfp", "" ,d)}"
 
TUNEVALID[callconvention-hard] = "Enable EABI hard float call convention, requires VFP."
-TUNE_CCARGS .= "${@bb.utils.contains("TUNE_FEATURES", "vfp", bb.utils.contains("TUNE_FEATURES", "callconvention-hard", " -mfloat-abi=hard", " -mfloat-abi=softfp", d), "" ,d)}"
-ARMPKGSFX_EABI .= "${@bb.utils.contains("TUNE_FEATURES", [ "callconvention-hard", "vfp" ], "hf", "", d)}"
+TUNE_CCARGS .= "${@bb.utils.contains("TUNE_FEATURES", "vfp", bb.utils.contains("TUNE_FEATURES", "callconvention-hard", " -mfloat-abi=soft", " -mfloat-abi=soft", d), "" ,d)}"
+ARMPKGSFX_EABI .= "${@bb.utils.contains("TUNE_FEATURES", [ "callconvention-hard", "vfp" ], "", "", d)}"
```

### 2.3. 关于 xxx CPU 是否支持 NEON 和 VFP
xxx CPU 的芯片手册上有明确写支持 NEON 和 VFP，但是不清楚如何使能 NEON，是否要在上电和通过特定配置使能？
通过 `cat /proc/cpuinfo` 可以看出芯片不支持 NEON 和 VFP，但是 /proc/cpuinfo 能完全体现 CPU features 吗？
PS：内核有明确打开了配置 `CONFIG_NEON=y` 与 `CONFIG_VFP=y`。

```
root@ops-xxx:~# cat /proc/cpuinfo
processor       : 0
model name      : ARMv7 Processor rev 0 (v7l)
BogoMIPS        : 1987.37
Features        : swp half thumb fastmult edsp tls
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x3
CPU part        : 0xc09
CPU revision    : 0
 
processor       : 1
model name      : ARMv7 Processor rev 0 (v7l)
BogoMIPS        : 1993.93
Features        : swp half thumb fastmult edsp tls
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x3
CPU part        : 0xc09
CPU revision    : 0
 
Hardware        : yyy CPU
Revision        : 0000
Serial          : 0000000000000000
```

通过查阅资料发现 NEON 应该是在系统运行时使能的：
> 内核在遇到第一个NEON指令时会产生一个Undefined Instruction的异常，这会让内核自动重启NEON协处理器，内核还可以在上下文切换时关闭NEON来省电。
> BY [《ARM平台NEON指令的编译和优化》](http://houh-1984.blog.163.com/blog/static/31127834201382694010808/)

以及：
> arm 论坛 "[How to enable Neon in cortex A8?](https://community.arm.com/thread/8409 )"
> Yes you can't enable it directly from user mode you have to have the privilege though the startup code for the process or an interrupt when such an instruction is first used should do that normally. For the actual instructions needed have you seen: [ARM Compiler armcc User Guide : 5.5 Enabling NEON and FPU for bare-metal](http://infocenter.arm.com/help/topic/com.arm.doc.dui0472k/chr1359124222192.html)

直接在 uboot 中修订？http://lists.denx.de/pipermail/u-boot/2015-January/201269.html 

**TODO** 后续再做验证。
FPU 基本只应用于图像处理、音视频处理，与我们无关，因此可以放心不用。

3. 编译问题及修订记录
-------------------------------
### 3.1. python xattr 在 x86 平台上直接使用交叉编译结果
问题 log：

```
NOTE: Preparing RunQueue
NOTE: Checking sstate mirror object availability (for 545 objects)
NOTE: Executing SetScene Tasks
NOTE: Executing RunQueue Tasks
ERROR: Function failed: do_compile (log file is located at /home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/temp/log.do_compile.14582)
ERROR: Logfile of failure stored in: /home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/temp/log.do_compile.14582
Log data follows:
| DEBUG: Executing shell function do_compile
| running build
| Traceback (most recent call last):
|   File "setup.py", line 67, in <module>
|     cmdclass={'build': cffi_build},
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/distutils/core.py", line 151, in setup
|     dist.run_commands()
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/distutils/dist.py", line 953, in run_commands
|     self.run_command(cmd)
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/distutils/dist.py", line 971, in run_command
|     cmd_obj.ensure_finalized()
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/distutils/cmd.py", line 109, in ensure_finalized
|     self.finalize_options()
|   File "setup.py", line 15, in finalize_options
|     from xattr.lib import ffi
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/xattr-0.8.0/xattr/__init__.py", line 12, in <module>
|     from .lib import (XATTR_NOFOLLOW, XATTR_CREATE, XATTR_REPLACE,
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/xattr-0.8.0/xattr/lib.py", line 596, in <module>
|     """, ext_package='xattr')
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/site-packages/cffi/api.py", line 424, in verify
|     lib = self.verifier.load_library()
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/site-packages/cffi/verifier.py", line 111, in load_library
|     return self._load_library()
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/site-packages/cffi/verifier.py", line 222, in _load_library
|     return self._vengine.load_library()
|   File "/home/sunyongfeng/workshop/ops-build.rj/build/tmp/sysroots/x86_64-linux/usr/lib/python2.7/site-packages/cffi/vengine_cpy.py", line 155, in load_library
|     raise ffiplatform.VerificationError(error)
| cffi.ffiplatform.VerificationError: importing '/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/xattr-0.8.0/xattr/__pycache__/_cffi__xf88a1d89x3f40c4a9.so': /home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/xattr-0.8.0/xattr/__pycache__/_cffi__xf88a1d89x3f40c4a9.so: wrong ELF class: ELFCLASS32
| ERROR: python setup.py build execution failed.
| WARNING: exit code 1 from a shell command.
| ERROR: Function failed: do_compile (log file is located at /home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/temp/log.do_compile.14582)
ERROR: Task 3879 (/home/sunyongfeng/workshop/ops-build.rj/yocto/openswitch/meta-foss-openswitch/recipes-devtools/python/python-xattr_0.8.0.bb, do_compile) failed with exit code '1'
NOTE: Tasks Summary: Attempted 2351 tasks of which 2295 didn't need to be rerun and 1 failed.
Waiting for 0 running tasks to finish:

Summary: 1 task failed:
  /home/sunyongfeng/workshop/ops-build.rj/yocto/openswitch/meta-foss-openswitch/recipes-devtools/python/python-xattr_0.8.0.bb, do_compile
Summary: There was 1 WARNING message shown.
Summary: There was 1 ERROR message shown, returning a non-zero exit code.
tools/Rules.make:216: recipe for target '_fs' failed
make: *** [_fs] Error 1
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ops-build.rj$ 
```

原因：见 [github issue](https://github.com/pyca/cryptography/issues/1325)
> I don't think it's a cffi issue. I tried to cross compile for a 32 bit system from a 64 bit system. cffi module cffi_backend.so compiled OK for the target (that is 32 bit). **The problem is that when cryptography tries to pre-cross compile (for 32 bit) cffi modules (_Cryptography_cffi*.so), it uses (host) 64 bit compiler flags and tries to load the 32 bit cross compiled _cffi_backend.so.** This is why you get "wrong ELF class: ELFCLASS32".
>  
> There needs to be some way to configure the C compiler flags from within the cryptography package while cross compiling the cffi modules. You can see how cffi module itself is allowing that in its setup.py file. Another python package I was able to configure the target c flags is python readline.
>  
> I got around this problem (dirty hack) by skipping the cffi modules pre-cross compilation process (Cryptography_cffi*.so) and copying (during the build time) those modules I compiled on the target system.

这种交叉编译时在 build 系统上直接使用编译给 host 的结果，很是蛋疼。

解决：
> Well I could get it working myself by following midicase comment on vincentbernat/snimpy.
1. build and install host cffi
2. build and install target packages that depend in cffi
3. build and install target cffi
I've created a detailed install instructions https://github.com/AndreMiras/km/wiki/CFFI-Cross-Compile.

依葫芦画瓢，也衔把 cffi 编译成 build 主机的结果，后续 ARM 设备上要用的话，再编译一个 ARM 架构的 .so。
优雅的做法是打 patch 改 xattr 编译 cffi 的配置文件，先编译成 build 机可用、再编译成 host 机可用的结果。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/xattr-0.8.0/xattr/__pycache__$ gcc -o _cffi__xf88a1d89x3f40c4a9.so _cffi__xf88a1d89x3f40c4a9.c -shared -I/usr/include/python2.7/ -fPIC
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/python-xattr/0.8.0-r0/xattr-0.8.0/xattr/__pycache__$ ls
_cffi__x7c9e2f59xb862c7dd.c  _cffi__xf88a1d89x3f40c4a9.c  _cffi__xf88a1d89x3f40c4a9.so  _cffi__xf88a1d89x3f40c4a9.so.arm  xattr
```

### 3.2. go 编译失败
go 编译不过，go 的依赖方是 ops-webui，应该是 web 操作界面相关的东西，目前还用不上直接删了。

```
| WARNING: exit code 2 from a shell command.
| ERROR: Function failed: do_compile (log file is located at /home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/x86_64-openswitch-linux-gnueabi/go-cross/1.6.3-r0/temp/log.do_compile.31635)
ERROR: Task 3543 (/home/sunyongfeng/workshop/ops-build.rj/yocto/openswitch/meta-foss-openswitch/recipes-devtools/go/go-cross_1.6.3.bb, do_compile) failed with exit code '1'
NOTE: Tasks Summary: Attempted 3606 tasks of which 3599 didn't need to be rerun and 1 failed.
Waiting for 0 running tasks to finish:
 
Summary: 1 task failed:
```

* 删除 ops-webui.bb
* 删除 recipes-go
* 删除 recipes-nodejs

```
        deleted:    yocto/openswitch/meta-distro-openswitch/recipes-ops/mgmt/ops-webui.bb
        deleted:    yocto/openswitch/meta-foss-openswitch/recipes-go/prometheus/prometheus-promu_git.bb
        deleted:    yocto/openswitch/meta-foss-openswitch/recipes-nodejs/nodejs/nodejs.inc
        deleted:    yocto/openswitch/meta-foss-openswitch/recipes-nodejs/nodejs/nodejs4.inc
        deleted:    yocto/openswitch/meta-foss-openswitch/recipes-nodejs/nodejs/nodejs_4.1.2.bb
```

### 3.3. ops-* 多地方出现 VLOG format 和指针强转 Warning 导致编译失败
这纯粹是 ops 代码的问题，估计没有 porting 到 32 位机上。
出现 warning 的位置实在太多，暂不考虑直接改 .c/.h 源代码。考虑从编译选项入手。

由于带 `-Werror`，所以 Warning 会被当成 Error 处理。在 `local.conf` 中配置 `-Wno-error` 可体现在编译选项中，但是这个编译选项在 `-Werror` 之前，经确认，要放在之后才能生效。
因此需要考虑打 patch 删除源码配置文件中编译选项带的 `-Werror`。

ops-* 组件使用两种编译工具，CMake 与 autoconf。这里为快速适配，还没有使用打 patch 的方法打补丁。

#### 3.3.1. CMake 类处理
查找所有 `CMakeLists.txt` 文件，删除文件中的所有 `-Werror`。
```
find . -name "CMakeLists.txt" | xargs sed -i "s/ -Werror//g"
```

#### 3.3.2. autoconf 类处理
只有 `ops-lldpd` 组件用的 autoconf 且出现 warning。

*  删除 `build/tmp/stamps` 目录内 `ops-lldpd.do_configure.xxx`，这样做才能重新 `configure`；
* 修改 `build/tmp/work/armv4-openswitch-linux-gnueabi/ops-lldpd/gitAUTOINC+fd74e10ef2-r0/git/ 源码 `configure.ac` 文件，将其中的 `-Werror` 选项删除。

```
diff --git a/configure.ac b/configure.ac
index c476286..ea5d5fb 100644
--- a/configure.ac
+++ b/configure.ac
@@ -29,7 +29,7 @@ AC_CONFIG_MACRO_DIR([m4])
 AC_SUBST([CONFIGURE_ARGS], [$ac_configure_args])
 
 # Configure automake
-AM_INIT_AUTOMAKE([foreign -Wall -Werror])
+AM_INIT_AUTOMAKE([foreign -Wall])
 AM_MAINTAINER_MODE
 m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES(yes)])
 
@@ -70,7 +70,6 @@ AX_CFLAGS_GCC_OPTION([-fdiagnostics-show-option])
 AX_CFLAGS_GCC_OPTION([-fdiagnostics-color=auto])
 AX_CFLAGS_GCC_OPTION([-pipe])
 AX_CFLAGS_GCC_OPTION([-Wall])
-AX_CFLAGS_GCC_OPTION([-Werror])
 AX_CFLAGS_GCC_OPTION([-W])
 AX_CFLAGS_GCC_OPTION([-Wextra])
 AX_CFLAGS_GCC_OPTION([-Wformat])
```

这个定义没有的使用地方没有写好，导致好多 ops-lldpd 强转的地方编译不过：

```
104     /*                                                                                              
105      *  For the initial release, this will just refer to the                                        
106      *  relevant UCD header files.                                                                  
107      *    In due course, the types and structures relevant to the                                   
108      *  Net-SNMP API will be identified, and defined here directly.                                 
109      *                                                                                              
110      *  But for the time being, this header file is primarily a placeholder,                        
111      *  to allow application writers to adopt the new header file names.                            
112      */                                                                                             
113                                                                                                     
114 typedef union {                                                                                     
115    long           *integer;                                                                         
116    u_char         *string;                                                                          
117    oid            *objid;                                                                           
118    u_char         *bitstring;                                                                       
119    struct counter64 *counter64;                                                                     
120 #ifdef NETSNMP_WITH_OPAQUE_SPECIAL_TYPES                                                            
121    float          *floatVal;                                                                        
122    double         *doubleVal;                                                                       
123    /*                                                                                               
124     * t_union *unionVal;                                                                            
125     */                                                                                              
126 #endif                          /* NETSNMP_WITH_OPAQUE_SPECIAL_TYPES */                             
127 } netsnmp_vardata;                                                                                  
128                                                                                                     
129                                                                                                     
130 #define MAX_OID_LEN     128 /* max subid's in an oid */                                             
131                                                                                                     
132 /** @typedef struct variable_list netsnmp_variable_list                                             
133  * Typedefs the variable_list struct into netsnmp_variable_list */                                  
134 /** @struct variable_list                                                                           
135  * The netsnmp variable list binding structure, it's typedef'd to                                   
136  * netsnmp_variable_list.                                                                           
137  */                                                                                                 
138 typedef struct variable_list {                                                                      
139    /** NULL for last variable */                                                                    
140    struct variable_list *next_variable;                                                             
141    /** Object identifier of variable */                                                             
142    oid            *name;                                                                            
  ⮀ NORMAL ⮀  net-snmp/types.h  
```

log：

```
/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/ops-lldpd/gitAUTOINC+fd74e10ef2-r0/git/src/snmp/lldpPortConfigTable_interface.c: In function '_lldpPortConfigTable_get_column':
/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/ops-lldpd/gitAUTOINC+fd74e10ef2-r0/git/src/snmp/lldpPortConfigTable_interface.c:323:56: warning: cast increases required alignment of target type [-Wcast-align]
         rc = lldpPortConfigAdminStatus_get(rowreq_ctx, (long *)var->val.string);
                                                        ^
/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/ops-lldpd/gitAUTOINC+fd74e10ef2-r0/git/src/snmp/lldpPortConfigTable_interface.c:329:51: warning: cast increases required alignment of target type [-Wcast-align]
                                                   (long *)var->val.string);
                                                   ^
/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/ops-lldpd/gitAUTOINC+fd74e10ef2-r0/git/src/snmp/lldpPortConfigTable_interface.c:336:45: warning: cast increases required alignment of target type [-Wcast-align]
                                             (u_long *)var->val.string);
                                             ^
/home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/cortexa9-vfp-neon-openswitch-linux-gnueabi/ops-lldpd/gitAUTOINC+fd74e10ef2-r0/git/src/snmp/lldpPortConfigTable_interface.c:340:19: warning: cast increases required alignment of target type [-Wcast-align]
             if (*((u_long *)var->val.string) & mask)
```

### 3.4. openswitch-disk-image.bb task `do_rootfs` 失败
原因：usermod 命令找不到 group ovsdb-client 等 group。log：
```
| NOTE: Performing usermod with [-R /home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/xxx-openswitch-linux-gnueabi/openswitch-disk-image/1.0-r0/rootfs -g ovsdb-client opsd] and 1 times of retry
| usermod: group 'ovsdb-client' does not exist
| Server refused shutdown.  Remaining client fds: 2
| Client pids: 2799 8830
| Server will shut down after all clients exit.
| WARNING: usermod command did not succeed. Retrying...
| ERROR: Tried running usermod command 1 times without success, giving up
| WARNING: exit code 1 from a shell command.
| DEBUG: Python function do_rootfs finished
| ERROR: Function failed: set_user_group (log file is located at /home/sunyongfeng/workshop/ops-build.rj/build/tmp/work/xxx-openswitch-linux-gnueabi/openswitch-disk-image/1.0-r0/temp/log.do_rootfs.2799)
ERROR: Task 7 (/home/sunyongfeng/workshop/ops-build.rj/yocto/openswitch/meta-distro-openswitch/recipes-core/images/openswitch-disk-image.bb, do_rootfs) failed with exit code '1'
NOTE: Tasks Summary: Attempted 4408 tasks of which 4407 didn't need to be rerun and 1 failed.
No currently running tasks (4405 of 4411)

Summary: 1 task failed:
  /home/sunyongfeng/workshop/ops-build.rj/yocto/openswitch/meta-distro-openswitch/recipes-core/images/openswitch-disk-image.bb, do_rootfs
Summary: There was 1 WARNING message shown.
Summary: There was 1 ERROR message shown, returning a non-zero exit code.
tools/Rules.make:216: recipe for target '_fs' failed
make: *** [_fs] Error 1
```

看 rootfs 里面，没有看到 group 没有 ops_admin、ovsdb-client 等。而  `yocto/openswitch/meta-distro-openswitch/classes/openswitch-image.bbclass` 中的 EXTRA_USER_PARAMS 有添加用户到这几个 group 中。

```
inherit core-image extrausers 
EXTRA_USERS_PARAMS = "\ 
         useradd -N -M -r opsd; \ 
         usermod -s /bin/false opsd;\ 
         useradd -N -P netop netop; \ 
         useradd -N -P admin admin; \ 
         useradd -N -P remote_user remote_user; \   
         usermod -g ops_admin admin;\ 
         usermod -g ops_netop netop;\
         usermod -g ops_netop remote_user;\ 
         usermod -g ovsdb-client opsd;\ 
         usermod -G ovsdb-client netop;\ 
         usermod -G ovsdb-client remote_user;\
         usermod -s /bin/bash admin;\ 
         usermod -s /usr/bin/vtysh netop;\ 
         usermod -s /usr/bin/vtysh remote_user;\
         " 
IMAGE_FEATURES += "ssh-server-openssh" 
IMAGE_FEATURES += "package-management"

IMAGE_GEN_DEBUGFS = "1" 
IMAGE_FSTYPES += "dbg.tar" 
```

而 ` yocto/openswitch/meta-distro-openswitch/recipes-ops/openvswitch/ops-openvswitch.bb` 中有配置添加 group：

```
GROUPADD_PARAM_${PN} ="-g 1020 ovsdb-client;ops_netop;ops_admin"
```

这个配置没在 rootsf 中生效：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ops-build.rj/build/tmp/work/xxx-openswitch-linux-gnueabi/openswitch-disk-image/1.0-r0/rootfs$ cat etc/group 
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
input:x:19:
dialout:x:20:
fax:x:21:
voice:x:22:
cdrom:x:24:
floppy:x:25:
tape:x:26:
sudo:x:27:
audio:x:29:
dip:x:30:
www-data:x:33:
backup:x:34:
operator:x:37:
list:x:38:
irc:x:39:
src:x:40:
gnats:x:41:
shadow:x:42:
utmp:x:43:
video:x:44:
sasl:x:45:
plugdev:x:46:
staff:x:50:
games:x:60:
shutdown:x:70:
users:x:100:
nogroup:x:65534:
rpc:x:999:
netdev:x:998:
messagebus:x:997:
sshd:x:996:
lock:x:995:
systemd-journal:x:994:
systemd-journal-gateway:x:993:
systemd-timesync:x:992:
```

因此，在usermod 使用不存在的 group 前，先执行 groupadd。

```
diff --git a/yocto/openswitch/meta-distro-openswitch/classes/openswitch-image.bbclass b/yocto/openswitch/meta-distro-openswitch/classes/openswitch-image.bbclass
index 996503c..5e1e9e8 100644
--- a/yocto/openswitch/meta-distro-openswitch/classes/openswitch-image.bbclass
+++ b/yocto/openswitch/meta-distro-openswitch/classes/openswitch-image.bbclass
@@ -2,6 +2,9 @@ inherit core-image extrausers
 EXTRA_USERS_PARAMS = "\
          useradd -N -M -r opsd; \
          usermod -s /bin/false opsd;\
+         groupadd -g 1020 ovsdb-client; \
+         groupadd ops_netop; \
+         groupadd ops_admin; \
          usermod -g ovsdb-client opsd;\
          useradd -N -P netop netop; \
          useradd -N -P admin admin; \
```

4. 编译太慢？
------------------
### 4.1. 下载源码慢的问题
编译的时候，会实时下载要编译的源代码包，由于 GFW 的缘故，代码下载得很慢，导致整个工程编译时间超长。因此可一次下载，多次使用，改 `DL_DIR`。

* 原始位置，`tools/config/local.conf.in` 和 `tools/config/site.conf.in`
* 在 `make configure xxx` 之后，亦可直接修订 `build/conf/local.conf`

### 4.2. 如果使用已编译的结果
利用 yocto sstate 特性。配置 `SSTATE_DIR`

4.1. 和 4.2. 的配置修订如下：

```
diff --git a/tools/config/local.conf.in b/tools/config/local.conf.in
index 80eefbe..eaeb301 100644
--- a/tools/config/local.conf.in
+++ b/tools/config/local.conf.in
@@ -39,7 +39,7 @@ MACHINE = "##PLATFORM##"
 #
 # The default is a downloads directory under TOPDIR which is the build directory.
 #
-#DL_DIR ?= "${TOPDIR}/downloads"
+DL_DIR ?= "/home/sunyongfeng/backup/downloads"
 
 #
 # Where to place shared-state files
@@ -55,7 +55,7 @@ MACHINE = "##PLATFORM##"
 #
 # The default is a sstate-cache directory under TOPDIR.
 #
-#SSTATE_DIR ?= "${TOPDIR}/sstate-cache"
+SSTATE_DIR ?= "/home/sunyongfeng/backup/sstate-cache"
 
 #
 # Where to place the build output
@@ -197,10 +197,14 @@ BB_DISKMON_DIRS = "\
 # NOTE: if the mirror uses the same structure as SSTATE_DIR, you need to add PATH
 # at the end as shown in the examples below. This will be substituted with the
 # correct path within the directory structure.
+#SSTATE_MIRRORS ?= "\
+#file://.* http://##DISTRO_SSTATE_ADDRESS##/PATH \
+#"
 SSTATE_MIRRORS ?= "\
-file://.* http://##DISTRO_SSTATE_ADDRESS##/PATH"
+file:///home/sunyongfeng/backup/sstate-cache http://##DISTRO_SSTATE_ADDRESS##/PATH"
 
-SOURCE_MIRROR_URL ?= "http://##DISTRO_ARCHIVE_ADDRESS##/"
+#SOURCE_MIRROR_URL ?= "http://##DISTRO_ARCHIVE_ADDRESS##/"
+SOURCE_MIRROR_URL ?= "file:///home/sunyongfeng/backup/downloads/"
 INHERIT += "own-mirrors" 
 BB_GENERATE_MIRROR_TARBALLS = "1"

diff --git a/tools/config/site.conf.in b/tools/config/site.conf.in
index 9fa46c5..4aa4206 100644
--- a/tools/config/site.conf.in
+++ b/tools/config/site.conf.in
@@ -21,5 +21,5 @@ SCONF_VERSION = "1"
 ##ALL_PROXY##
 
 # Uncomment this to use a shared download directory
-#DL_DIR = "/some/shared/download/directory/"
+DL_DIR = "/home/sunyongfeng/backup/downloads/"
```

### 4.3. parse recipe 实在太慢
改 ops-* 的源码路径为 github。但是 ops-tunnel github 上面还没有，需要自己改 DL_DIR 中的 ops-tunnel 包名。

```
DL_DIR 请改成你自己的 DL_DIR

mv DL_DIR/git2/git.openswitch.net.openswitch.ops-tunnel.done DL_DIR/git2/github.com.open-switch.ops-tunnel.done
mv DL_DIR/git2/git.openswitch.net.openswitch.ops-tunnel DL_DIR/git2/github.com.open-switch.ops-tunnel
```

patch：

```
diff --git a/yocto/openswitch/meta-distro-openswitch/conf/distro/openswitch.conf b/yocto/openswitch/meta-distro-openswitch/conf/distro/openswitch.conf
index dd776f0..b6c18b3 100644
--- a/yocto/openswitch/meta-distro-openswitch/conf/distro/openswitch.conf
+++ b/yocto/openswitch/meta-distro-openswitch/conf/distro/openswitch.conf
@@ -6,8 +6,11 @@ DISTRO_CODENAME = "epazote"
 SDK_VENDOR = "-openswitchsdk"
 SDK_VERSION := "${@'${DISTRO_VERSION}'.replace('snapshot-${DATE}','snapshot')}"
 
-OPS_REPO_HOSTNAME ?= "git.openswitch.net"
-OPS_REPO_PATH ?= "openswitch"
+#OPS_REPO_HOSTNAME ?= "git.openswitch.net"
+#OPS_REPO_PATH ?= "openswitch"
+#OPS_REPO_BASE_URL ?= "git://${OPS_REPO_HOSTNAME}/${OPS_REPO_PATH}"
+OPS_REPO_HOSTNAME ?= "github.com"
+OPS_REPO_PATH ?= "open-switch"
 OPS_REPO_BASE_URL ?= "git://${OPS_REPO_HOSTNAME}/${OPS_REPO_PATH}"
 OPS_REPO_PROTOCOL ?= "https"
 OPS_REPO_BRANCH ?= "master"
diff --git a/yocto/openswitch/meta-distro-openswitch/recipes-ops/infra/ops-website.bb b/yocto/openswitch/meta-distro-openswitch/recipes-ops/infra/ops-website.bb
index 79c50f6..caa166d 100644
--- a/yocto/openswitch/meta-distro-openswitch/recipes-ops/infra/ops-website.bb
+++ b/yocto/openswitch/meta-distro-openswitch/recipes-ops/infra/ops-website.bb
@@ -4,7 +4,7 @@ LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/Apache-2.0;md5=89aea4e17d99a7ca
 
 BRANCH ?= "${OPS_REPO_BRANCH}"
 
-SRC_URI = "git://${OPS_REPO_HOSTNAME}/infra/website;protocol=${OPS_REPO_PROTOCOL};branch=${BRANCH} \
+SRC_URI = "git://${OPS_REPO_HOSTNAME}/open-switch/infra_website;protocol=${OPS_REPO_PROTOCOL};branch=${BRANCH} \
 "
 
 SRCREV="${AUTOREV}"
diff --git a/yocto/openswitch/meta-distro-openswitch/recipes-ops/switchd/ops-switchd.bb b/yocto/openswitch/meta-distro-openswitch/recipes-ops/switchd/ops-switchd.bb
index 8d622bd..d74cd9e 100644
--- a/yocto/openswitch/meta-distro-openswitch/recipes-ops/switchd/ops-switchd.bb
+++ b/yocto/openswitch/meta-distro-openswitch/recipes-ops/switchd/ops-switchd.bb
@@ -6,7 +6,7 @@ DEPENDS = "ops ops-openvswitch ops-ovsdb ops-utils libyaml jemalloc ops-cli"
 
 BRANCH ?= "${OPS_REPO_BRANCH}"
 
-SRC_URI = "${OPS_REPO_BASE_URL}/ops-switchd;protocol=${OPS_REPO_PROTOCOL};branch=${BRANCH} \
+SRC_URI = "${OPS_REPO_BASE_URL}/ops-switchd.git;protocol=${OPS_REPO_PROTOCOL};branch=${BRANCH} \
    file://switchd_nnn.service \
    file://switchd_sim.service \
    file://switchd_p4sim.service \
@@ -14,7 +14,7 @@ SRC_URI = "${OPS_REPO_BASE_URL}/ops-switchd;protocol=${OPS_REPO_PROTOCOL};branch
    file://switchd_sai.service \
 "
 
-SRCREV = "f17ef096d728b79c77a24cd908fdbcf9b18bcd6c"
+SRCREV = "d345f87cd17ced3b97a73ace23c387de5db62842"
 
 # When using AUTOREV, we need to force the package version to the revision of git
 # in order to avoid stale shared states.
@@ -33,18 +33,18 @@ FILES_${PN} = "${sbindir}/ops-switchd ${libdir}/libswitchd_plugins.so.1* ${libdi
 FILES_${PN} += "/usr/lib/cli/plugins/"
 
 do_install_append() {
-   install -d ${D}${systemd_unitdir}/system
-   if ${@bb.utils.contains('MACHINE_FEATURES','nnnmmm','true','false',d)}; then
-      install -m 0644 ${WORKDIR}/switchd_nnn.service ${D}${systemd_unitdir}/system/switchd.service
-   elif ${@bb.utils.contains('MACHINE_FEATURES','xpliant','true','false',d)}; then
-      install -m 0644 ${WORKDIR}/switchd_xpliant.service ${D}${systemd_unitdir}/system/switchd.service
-   elif ${@bb.utils.contains('IMAGE_FEATURES','ops-p4','true','false',d)}; then
+    install -d ${D}${systemd_unitdir}/system
+#if ${@bb.utils.contains('MACHINE_FEATURES','nnnmmm','true','false',d)}; then
+#      install -m 0644 ${WORKDIR}/switchd_nnn.service ${D}${systemd_unitdir}/system/switchd.service
+#   elif ${@bb.utils.contains('MACHINE_FEATURES','xpliant','true','false',d)}; then
+#      install -m 0644 ${WORKDIR}/switchd_xpliant.service ${D}${systemd_unitdir}/system/switchd.service
+#   elif ${@bb.utils.contains('IMAGE_FEATURES','ops-p4','true','false',d)}; then
       install -m 0644 ${WORKDIR}/switchd_p4sim.service ${D}${systemd_unitdir}/system/switchd.service
-   elif ${@bb.utils.contains('MACHINE_FEATURES','ops-container','true','false',d)}; then
-      install -m 0644 ${WORKDIR}/switchd_sim.service ${D}${systemd_unitdir}/system/switchd.service
-   elif ${@bb.utils.contains('MACHINE_FEATURES','ops-sai','true','false',d)}; then
-      install -m 0644 ${WORKDIR}/switchd_sai.service ${D}${systemd_unitdir}/system/switchd.service
-   fi
+#   elif ${@bb.utils.contains('MACHINE_FEATURES','ops-container','true','false',d)}; then
+#      install -m 0644 ${WORKDIR}/switchd_sim.service ${D}${systemd_unitdir}/system/switchd.service
+#   elif ${@bb.utils.contains('MACHINE_FEATURES','ops-sai','true','false',d)}; then
+#      install -m 0644 ${WORKDIR}/switchd_sai.service ${D}${systemd_unitdir}/system/switchd.service
+#   fi
 
    install -d ${D}/usr/share/opsplugins
    for plugin in $(find ${S}/opsplugins -name "*.py"); do \
```

5. 其他配置
----------------
### 5.1. 如何新增 CFLAG、LDFLAG 选项？
在 local.conf 中添加对应 FLAG，以 -Wno-error 为例。

```
BUILD_CFLAGS += " -Wno-error "
BUILD_CPPFLAGS += " -Wno-error "
BUILD_CXXFLAGS += " -Wno-error "
TARGET_CFLAGS += " -Wno-error "
TARGET_CPPFLAGS += " -Wno-error "
TARGET_CXXFLAGS += " -Wno-error "
```

### 5.2. 使用 pre-built 交叉编译工具链（linaro）
FAIL。

可研究 SDK，将交叉编译链一次编出，后续无需再编译。

### 5.3. 配置 `ops-switchd-p4switch-plugin`
修改 machine 的 conf，`yocto/openswitch/meta-platform-openswitch-xxx/conf/machine/xxx.conf `：

```
PREFERRED_PROVIDER_virtual/ops-switchd-switch-api-plugin ?= "ops-switchd-p4switch-plugin"
```

官方 README 的说明应已过时：
> The P4 plugin is activated adding the following line in /build/conf/local.conf EXTRA_IMAGE_FEATURES = "ops-p4" (use += if other image features are defined)
> https://github.com/open-switch/ops-switchd-p4switch-plugin

6. 运行问题记录与解决
-------------------------------
### 6.1. ops-switchd-p4switch-plugin 没有打包到 rootfs

### 6.2. 运行时出现 rsyslog、ops-init.servic 等业务起不来

log 如下：

```
[20161110:184441.388][FAILED] Failed to start openswitch first boot process.
[20161110:184441.388]See "systemctl status ops-first-boot.service" for details.
[20161110:184445.440][FAILED] Failed to start OpenSwitch system initialization script.
[20161110:184445.440]See "systemctl status ops-init.service" for details.
[20161110:184447.615]systemd[1]: Unit ops-powerd.service entered failed state.
[20161110:184447.615]systemd[1]: ops-powerd.service failed.
[20161110:184447.646]systemd[1]: Unit ops-lacpd.service entered failed state.
[20161110:184447.646]systemd[1]: ops-lacpd.service failed.
[20161110:184447.677]systemd[1]: Unit ops-portd.service entered failed state.
[20161110:184447.677]systemd[1]: ops-portd.service failed.
[20161110:184447.708]systemd[1]: Unit ops-tempd.service entered failed state.
[20161110:184447.708]systemd[1]: ops-tempd.service failed.
[20161110:184447.740]systemd[1]: Unit ops-zebra.service entered failed state.
[20161110:184447.740]systemd[1]: ops-zebra.service failed.
[20161110:184447.771]systemd[1]: Unit ops-ledd.service entered failed state.
[20161110:184447.771]systemd[1]: ops-ledd.service failed.
[20161110:184447.802]systemd[1]: Unit aaautils.service entered failed state.
[20161110:184447.802]systemd[1]: aaautils.service failed.
[20161110:184447.833]systemd[1]: Unit ops-stpd.service entered failed state.
[20161110:184447.833]systemd[1]: ops-stpd.service failed.
[20161110:184447.865]systemd[1]: Unit ops-bgpd.service entered failed state.
[20161110:184447.865]systemd[1]: ops-bgpd.service failed.
[20161110:184447.865]systemd[1]: Unit ops-classifierd.service entered failed state.
[20161110:184447.865]systemd[1]: ops-classifierd.service failed.
[20161110:184447.927]systemd[1]: Unit ops-relay.service entered failed state.
[20161110:184447.927]systemd[1]: ops-relay.service failed.
[20161110:184447.958]systemd[1]: Unit ops-intfd.service entered failed state.
[20161110:184447.958]systemd[1]: ops-intfd.service failed.
[20161110:184447.990]systemd[1]: Unit ops-ospfd.service entered failed state.
[20161110:184447.990]systemd[1]: ops-ospfd.service failed.
[20161110:184448.021]systemd[1]: Unit ops-arpmgrd.service entered failed state.
[20161110:184448.021]systemd[1]: ops-arpmgrd.service failed.
[20161110:184448.052]systemd[1]: Unit dhcp_tftp.service entered failed state.
[20161110:184448.052]systemd[1]: dhcp_tftp.service failed.
[20161110:184448.083]systemd[1]: Unit ops-fand.service entered failed state.
[20161110:184448.083]systemd[1]: ops-fand.service failed.
[20161110:184448.115]systemd[1]: Unit ops-sysd.service entered failed state.
[20161110:184448.115]systemd[1]: ops-sysd.service failed.
[20161110:184448.146]systemd[1]: Unit ops-lldpd.service entered failed state.
[20161110:184448.146]systemd[1]: ops-lldpd.service failed.
[20161110:184448.146]systemd[1]: Unit ops-vland.service entered failed state.
[20161110:184448.146]systemd[1]: ops-vland.service failed.
```

进一步看 log：所有问题都是 `open("/proc/self/ns/net"): No such file or directory`，原因是内核配置 namespace 没有开启。

```
[20161110:184548.411]ops-xxx:~$ systemctl status ops-first-boot.service
[20161110:184548.536]● ops-first-boot.service - openswitch first boot process
[20161110:184548.536]   Loaded: loaded (/lib/systemd/system/ops-first-boot.service; enabled; vendor preset: enabled)
[20161110:184548.536]   Active: failed (Result: exit-code) since Thu 1970-01-01 00:00:08 UTC; 1min 7s ago
[20161110:184548.536]  Process: 100 ExecStart=/usr/sbin/setcap cap_audit_write+eip /usr/bin/vtysh (code=exited, status=1/FAILURE)
[20161110:184548.551] Main PID: 100 (code=exited, status=1/FAILURE)
[20161110:184549.505]ops-xxx:~$ 
[20161110:184613.881]ops-xxx:~$ systemd[1]: Unit rsyslog.service entered failed state.
[20161110:184622.878]systemd[1]: Unit rsyslog.service entered failed state.
[20161110:184622.878]systemd[1]: rsyslog.service failed.
[20161110:184624.488]systemctl status rsyslog.service
[20161110:184624.534]● rsyslog.service - System Logging Service
[20161110:184624.534]   Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
[20161110:184624.534]   Active: failed (Result: exit-code) since Thu 1970-01-01 00:01:51 UTC; 41ms ago
[20161110:184624.534]  Process: 3112 ExecStart=/sbin/ip netns exec swns /usr/sbin/rsyslogd -n (code=exited, status=1/FAILURE)
[20161110:184624.550] Main PID: 3112 (code=exited, status=1/FAILURE)
[20161110:184830.707]ops-xxx:~$ systemctl status ops-init.service
[20161110:184830.770]● ops-init.service - OpenSwitch system initialization script
[20161110:184830.770]   Loaded: loaded (/lib/systemd/system/ops-init.service; enabled; vendor preset: enabled)
[20161110:184830.770]   Active: failed (Result: exit-code) since Thu 1970-01-01 00:00:11 UTC; 3min 46s ago
[20161110:184830.770]  Process: 161 ExecStart=/usr/sbin/ops-init (code=exited, status=1/FAILURE)
[20161110:184830.770] Main PID: 161 (code=exited, status=1/FAILURE)
```

对应的 namespace 内核配置：

```
CONFIG_NAMESPACES=y
CONFIG_UTS_NS=y
CONFIG_IPC_NS=y
# CONFIG_USER_NS is not set
CONFIG_PID_NS=y
CONFIG_NET_NS=y
CONFIG_UIDGID_CONVERTED=y
```

### 6.3. P4 交换机模拟器启动失败
log：

```
[FAILED] Failed to start P4 Switch simulation Daemon.
See "systemctl status simple_switch.service" for details.
```

原因：内核配置没有开启支持 VNET

```
ops-xxx:~$ systemctl status simple_switch.service
● simple_switch.service - P4 Switch simulation Daemon
   Loaded: loaded (/lib/systemd/system/simple_switch.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Thu 1970-01-01 00:00:13 UTC; 1min 26s ago
  Process: 222 ExecStartPre=/sbin/ip netns exec emulns ip link add veth250 type veth peer name veth251 (code=exited, status=2)
  Process: 214 ExecStartPre=/sbin/ip netns add emulns (code=exited, status=0/SUCCESS)
  Process: 210 ExecStartPre=/sbin/ip netns del emulns (code=exited, status=1/FAILURE)
  Process: 206 ExecStartPre=/bin/rm -f /var/run/simple_switch.pid (code=exited, status=0/SUCCESS)
ops-xxx:~$ sudo/sbin/ip netns exec emulns ip link add veth250 type veth peer name
RTNETLINK answers: Operation not supported
```

### 6.4. CLI 不可用
log：

```
root@ops-xxx:~# vtysh 
ops-xxx# show interface 
System is not ready. Please retry after few seconds..
```

ip netns exec emulns ip tuntap add mode tap eth0
ip netns exec emulns ifconfig eth0 up

查看 bmv2 runtime command line 命令：https://github.com/p4lang/behavioral-model/blob/master/tools/runtime_CLI.py
runtime cli：ip netns exec emulns /usr/sbin/simple_switch -i 64@veth250 --thrift-port 10001 --nanolog ipc:///tmp/bm-log.ipc /usr/share/ovs_p4_plugin/switch_bmv2.json

```
#!/bin/bash
set -x
 
for iface in `seq 1 7` ; do
  /sbin/ip netns exec emulns ip tuntap add mode tap eth$iface
  /sbin/ip netns exec emulns /sbin/ip ifconfig eth$iface up
  echo port_add eth$iface `expr $iface - 1` | /sbin/ip netns exec emulns /usr/bin/bm_tools/runtime_CLI.py --json /usr/share/ovs_p4_plugin/switch_bmv2.json --thrift-port 10001
done
```

可能需要配置的地方：
* https://github.com/open-switch/ops-build/blob/55c9f56fd3fc7a87cd5c7273374b910998630203/yocto/openswitch/meta-platform-openswitch-genericx86-64/recipes-ops/platform/ops-hw-config.bbappend
* https://github.com/open-switch/ops-hw-config/tree/master/Generic-x86/X86-64/P4
* https://github.com/open-switch/ops-build/blob/55c9f56fd3fc7a87cd5c7273374b910998630203/yocto/openswitch/meta-distro-openswitch/recipes-ops/openvswitch/ops-openvswitch.bb
* IMAGE_FEATURES[validitems] += "ops-p4"

7. 其他问题
----------------
### x86_64 上的一个 QA 问题
编译时提示：

```
WARNING: QA Issue: ops-cli rdepends on libcap, but it isn't a build dependency? [build-deps]
```

这个问题可能导致编译  ops-cli 时，configure 不过。
