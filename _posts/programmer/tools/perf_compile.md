
title: Linux Kernel 工具 perf 交叉编译
date: 2016-08-24 20:13:24
toc: true
tags: [perf]
categories: programmer
keywords: [linux, kernel, perf, compile, arm, mips, 性能, 分析, 工具, ubuntu, x86, callchains, zlib, failed, 编译, elfutils, 失败, ]
description: linux kernel perf 工具编译到 arm/mips 平台；ubuntu x86 平台使用 perf 工具的记录。
---

以下译自 [en.wikipedia](https://en.wikipedia.org/wiki/Perf_(Linux))
> **perf** （有时亦称为 "Perf Events" 或 perf 工具，原始名称为 "Performance Counters for linux", PCL），是一个 Linux 性能分析工具，从 Linux 内核版本 2.6.31 导入。用户空间控制工具名为 perf，在命令行使用，提供一系列子命令，适用于整个系统（包含用户空间、内核空间代码）的 profiling 数据统计（即性能剖析数据统计）。
> 
> perf 支持硬件性能计数器、tracepoints、软件性能计数器（如 hrtimer），和动态 probes （比如 kprobes 或uprobes）。IBM 在 2012 年将 perf 与 Oprofile 评为两个最常用的 linux performance counter profiling tools。

内核选项
--------

```
* General setup  ---> 
  + [*] Profiling support
  + Kernel Performance Events And Counters  --->
    -  [*] Kernel performance events and counters
    -  [*]   Debug: use vmalloc to back perf mmap() buffers
```

体现在配置文件上的配置项：

ARM Cortex a9：
* CONFIG_HAVE_PERF_EVENTS=y
* CONFIG_PERF_USE_VMALLOC=y
* CONFIG_PERF_EVENTS=y
* CONFIG_DEBUG_PERF_USE_VMALLOC=y
* CONFIG_VM_EVENT_COUNTERS=y
* CONFIG_PROFILING=y

MIPS OCTEON II：

* CONFIG_CAVIUM_OCTEON_PERF=y
* CONFIG_HAVE_PERF_EVENTS=y
* CONFIG_PERF_USE_VMALLOC=y
* CONFIG_PERF_EVENTS=y
* CONFIG_DEBUG_PERF_USE_VMALLOC=y
* CONFIG_VM_EVENT_COUNTERS=y
* CONFIG_PROFILING=y
* CONFIG_HAVE_PERF_REGS=y
* CONFIG_HAVE_PERF_USER_STACK_DUMP=y

X86_64
------
ubuntu 16.04 上：
* 安装 `linux-tools-generic`、`linux-cloud-tools-generic`、`perf-tools-unstable`
* 如果 perf 工具支持的内核不是当前默认启机的内核，则启机后，通过 grub 菜单选择对应的内核后，perf 才能使用。

实测可用。

MIPS OCTEON II
----
截止 2016年8月24日，MIPS 平台的 perf 还不支持用户空间 callchain，即 perf 使用时无法识别用户空间的函数。

来自 [arch/mips/kernel/perf_events.c](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/arch/mips/kernel/perf_event.c):
```c
 23 /*                                                                                                  
 24  * Leave userspace callchain empty for now. When we find a way to trace                             
 25  * the user stack callchains, we will add it here.                                                  
 26  */
```

实测不支持用户空间 callchains。

```
# perf version : 3.9.9.g3b9f02
# arch : mips64
# nrcpus online : 4
# nrcpus avail : 4
# cpudesc : Cavium Octeon II V0.1
# total memory : 4093516 kB
# cmdline : /bin/perf record -g -e cpu-clock ./t1 
# event : name = cpu-clock, type = 1, config = 0x0, config1 = 0x0, config2 = 0x0, excl_usr = 0, excl_kern = 0, excl_host = 0, excl_guest = 1, precise_ip = 0
# HEADER_CPU_TOPOLOGY info available, use -I to display
# pmu mappings: software = 1, uncore_l2c = 7, uncore_tad = 8, uncore_mc = 6
# ========
#
# Samples: 6K of event 'cpu-clock'
# Event count (approx.): 1560000000
#
# Overhead  Command      Shared Object
# ........  .......  .................
#
    99.92%       t1  t1 （用户空间程序）              

     0.06%       t1  [kernel.kallsyms]
                 |          
                 |--25.00%--kmem_cache_alloc
                 |          anon_vma_prepare
                 |          handle_pte_fault
                 |          __do_page_fault
                 |          ret_from_exception
                 |          
                 |--25.00%--__update_tlb
                 |          __do_fault
                 |          handle_pte_fault
                 |          __do_page_fault
                 |          ret_from_exception
                 |          
                 |--25.00%-- unlock_page
                 |          __do_fault
                 |          handle_pte_fault
                 |          __do_page_fault
                 |          ret_from_exception
                 |          
                  --25.00%--release_pages
                            pagevec_lru_move_fn
                            lru_add_drain_cpu
/data2 # 
```

ARM Cortex a9
---
实测perf stat 时 counter 无计数，perf record 亦无数据，还不清楚问题在哪。

```
~ # perf stat ls
bin            etc            mnt            rgos           sys
boot           home           perf.data      root           tmp
bootloader     include        perf.data.old  rootfs         usr
data           lib            proc           sbin           var
dev            linuxrc        rg_cfg         share

 Performance counter stats for 'ls':

     <not counted> task-clock              
     <not counted> context-switches        
     <not counted> cpu-migrations          
     <not counted> page-faults             
   <not supported> cycles                  
   <not supported> stalled-cycles-frontend 
   <not supported> stalled-cycles-backend  
   <not supported> instructions            
   <not supported> branches                
   <not supported> branch-misses           

       0.003733662 seconds time elapsed
~ #
```

以下为编译记录：

交叉编译记录参考，丫凡的博文《[交叉编译 perf for arm](http://freenix.blogcn.com/articles/%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-perf-for-arm.html)》。

* 内核版本：3.10.18
* arm 交叉编译工具链：arm-cortex_a9-linux-gnueabi-gcc
* 依赖：zlib、elfutils

### zlib
zlib 因公司的交叉编译环境已支持，不用重新编译，略过。

### elfutils 交叉编译记录
* 版本，0.150，[官方下载地址](https://fedorahosted.org/releases/e/l/elfutils/0.150/elfutils-0.150.tar.bz2)。

如果直接以 `./configure --host=arm-cotex_a9-linux-gnueabi --prefix=/home/sunnogo/workshop; make`，会编译不过，挂在：

```
arm-cortex_a9-linux-gnueabi-gcc -std=gnu99 -Wall -Wshadow -Werror -Wunused -Wextra -Wformat=2   -fpic -fdollars-in-identifiers -I/home/sunnogo/workshop/rgosm-build/public/include -I/home/sunnogo/workshop/rgosm-build/prj_s6000e/images/header -D_LITTLE_ENDIAN  -L/home/sunnogo/workshop/rgosm-build/prj_s6000e/images/lib -L/home/sunnogo/workshop/rgosm-build/prj_s6000e/images/rootfs/lib -o i386_gendis i386_gendis.o i386_lex.o i386_parse.o ../lib/libeu.a -lm  
./i386_gendis i386_defs > i386_dis.h
/bin/bash: ./i386_gendis: cannot execute binary file: 可执行文件格式错误
Makefile:535: recipe for target 'i386_dis.h' failed
make[2]: *** [i386_dis.h] Error 126
rm i386_defs
make[2]: Leaving directory '/home/sunyongfeng/workshop/elfutils-0.150/libcpu'
Makefile:348: recipe for target 'all-recursive' failed
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory '/home/sunyongfeng/workshop/elfutils-0.150'
Makefile:262: recipe for target 'all' failed
make: *** [all] Error 2
sunnogo@R04220:~/workshop/elfutils-0.150$ 
```

* 配置
  + aclocal
  + autoheader
  + autoconf
  + automake --add-missing 
  + ./configure --host=arm-cotex_a9-linux-gnueabi --prefix=/home/sunnogo/workshop

```
sunnogo@R04220:~/workshop/elfutils-0.150$ aclocal
configure.ac:66: warning: AC_LANG_CONFTEST: no AC_LANG_SOURCE call detected in body
../../lib/autoconf/lang.m4:193: AC_LANG_CONFTEST is expanded from...
../../lib/autoconf/general.m4:2601: _AC_COMPILE_IFELSE is expanded from...
../../lib/autoconf/general.m4:2617: AC_COMPILE_IFELSE is expanded from...
../../lib/m4sugar/m4sh.m4:639: AS_IF is expanded from...
../../lib/autoconf/general.m4:2042: AC_CACHE_VAL is expanded from...
../../lib/autoconf/general.m4:2063: AC_CACHE_CHECK is expanded from...
configure.ac:66: the top level
configure.ac:66: warning: AC_LANG_CONFTEST: no AC_LANG_SOURCE call detected in body
../../lib/autoconf/lang.m4:193: AC_LANG_CONFTEST is expanded from...
../../lib/autoconf/general.m4:2601: _AC_COMPILE_IFELSE is expanded from...
../../lib/autoconf/general.m4:2617: AC_COMPILE_IFELSE is expanded from...
../../lib/m4sugar/m4sh.m4:639: AS_IF is expanded from...
../../lib/autoconf/general.m4:2042: AC_CACHE_VAL is expanded from...
../../lib/autoconf/general.m4:2063: AC_CACHE_CHECK is expanded from...
configure.ac:66: the top level
sunnogo@R04220:~/workshop/elfutils-0.150$ 
sunnogo@R04220:~/workshop/elfutils-0.150$ autoheader 
configure.ac:66: warning: AC_LANG_CONFTEST: no AC_LANG_SOURCE call detected in body
../../lib/autoconf/lang.m4:193: AC_LANG_CONFTEST is expanded from...
../../lib/autoconf/general.m4:2601: _AC_COMPILE_IFELSE is expanded from...
../../lib/autoconf/general.m4:2617: AC_COMPILE_IFELSE is expanded from...
../../lib/m4sugar/m4sh.m4:639: AS_IF is expanded from...
../../lib/autoconf/general.m4:2042: AC_CACHE_VAL is expanded from...
../../lib/autoconf/general.m4:2063: AC_CACHE_CHECK is expanded from...
configure.ac:66: the top level
sunnogo@R04220:~/workshop/elfutils-0.150$ autoconf 
configure.ac:66: warning: AC_LANG_CONFTEST: no AC_LANG_SOURCE call detected in body
../../lib/autoconf/lang.m4:193: AC_LANG_CONFTEST is expanded from...
../../lib/autoconf/general.m4:2601: _AC_COMPILE_IFELSE is expanded from...
../../lib/autoconf/general.m4:2617: AC_COMPILE_IFELSE is expanded from...
../../lib/m4sugar/m4sh.m4:639: AS_IF is expanded from...
../../lib/autoconf/general.m4:2042: AC_CACHE_VAL is expanded from...
../../lib/autoconf/general.m4:2063: AC_CACHE_CHECK is expanded from...
configure.ac:66: the top level
sunnogo@R04220:~/workshop/elfutils-0.150$ automake --add-missing 
configure.ac:66: warning: AC_LANG_CONFTEST: no AC_LANG_SOURCE call detected in body
../../lib/autoconf/lang.m4:193: AC_LANG_CONFTEST is expanded from...
../../lib/autoconf/general.m4:2601: _AC_COMPILE_IFELSE is expanded from...
../../lib/autoconf/general.m4:2617: AC_COMPILE_IFELSE is expanded from...
../../lib/m4sugar/m4sh.m4:639: AS_IF is expanded from...
../../lib/autoconf/general.m4:2042: AC_CACHE_VAL is expanded from...
../../lib/autoconf/general.m4:2063: AC_CACHE_CHECK is expanded from...
configure.ac:66: the top level
configure.ac:242: warning: The 'AM_PROG_MKDIR_P' macro is deprecated, and its use is discouraged.
configure.ac:242: You should use the Autoconf-provided 'AC_PROG_MKDIR_P' macro instead,
configure.ac:242: and use '$(MKDIR_P)' instead of '$(mkdir_p)'in your Makefile.am files.
configure.ac:61: installing 'config/compile'
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
backends/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
lib/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
libasm/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
libcpu/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
libdw/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
libdwfl/Makefile.am:29:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
libebl/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
libelf/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
src/Makefile.am:27:   'config/eu.am' included from here
config/eu.am:29: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
tests/Makefile.am:27:   'config/eu.am' included from here
parallel-tests: installing 'config/test-driver'
sunnogo@R04220:~/workshop/elfutils-0.150$ 
```

* 修改 Makefile
  + 删除根目录 Makefile 中的 libcpu 
  + 删除 backends/Makefile 中的 libebl_i386.a and libebl_x86_64.a 相关内容 (这里把所有 i386 和 x86_64 字眼所在行全部注释)
  + test 代码依赖 libz
  
patch 如下：
```patch
diff -uNr elfutils-0.150/backends/Makefile elfutils-0.150.change_makefile/backends/Makefile
--- elfutils-0.150/backends/Makefile	2016-08-24 14:28:43.122364093 +0800
+++ elfutils-0.150.change_makefile/backends/Makefile	2016-08-24 10:44:13.750042768 +0800
@@ -123,13 +123,13 @@
 	arm_regs.$(OBJEXT) arm_corenote.$(OBJEXT) arm_auxv.$(OBJEXT) \
 	arm_attrs.$(OBJEXT) arm_retval.$(OBJEXT)
 libebl_arm_pic_a_OBJECTS = $(am_libebl_arm_pic_a_OBJECTS)
-libebl_i386_pic_a_AR = $(AR) $(ARFLAGS)
-libebl_i386_pic_a_LIBADD =
-am__objects_3 = i386_init.$(OBJEXT) i386_symbol.$(OBJEXT) \
+#libebl_i386_pic_a_AR = $(AR) $(ARFLAGS)
+#libebl_i386_pic_a_LIBADD =
+#am__objects_3 = i386_init.$(OBJEXT) i386_symbol.$(OBJEXT) \
 	i386_corenote.$(OBJEXT) i386_cfi.$(OBJEXT) \
 	i386_retval.$(OBJEXT) i386_regs.$(OBJEXT) i386_auxv.$(OBJEXT) \
 	i386_syscall.$(OBJEXT)
-libebl_i386_pic_a_OBJECTS = $(am_libebl_i386_pic_a_OBJECTS)
+#libebl_i386_pic_a_OBJECTS = $(am_libebl_i386_pic_a_OBJECTS)
 libebl_ia64_pic_a_AR = $(AR) $(ARFLAGS)
 libebl_ia64_pic_a_LIBADD =
 am__objects_4 = ia64_init.$(OBJEXT) ia64_symbol.$(OBJEXT) \
@@ -165,13 +165,13 @@
 	sparc_corenote.$(OBJEXT) sparc64_corenote.$(OBJEXT) \
 	sparc_auxv.$(OBJEXT)
 libebl_sparc_pic_a_OBJECTS = $(am_libebl_sparc_pic_a_OBJECTS)
-libebl_x86_64_pic_a_AR = $(AR) $(ARFLAGS)
-libebl_x86_64_pic_a_LIBADD =
-am__objects_10 = x86_64_init.$(OBJEXT) x86_64_symbol.$(OBJEXT) \
+#libebl_x86_64_pic_a_AR = $(AR) $(ARFLAGS)
+#libebl_x86_64_pic_a_LIBADD =
+#am__objects_10 = x86_64_init.$(OBJEXT) x86_64_symbol.$(OBJEXT) \
 	x86_64_corenote.$(OBJEXT) x86_64_cfi.$(OBJEXT) \
 	x86_64_retval.$(OBJEXT) x86_64_regs.$(OBJEXT) \
 	i386_auxv.$(OBJEXT) x86_64_syscall.$(OBJEXT)
-libebl_x86_64_pic_a_OBJECTS = $(am_libebl_x86_64_pic_a_OBJECTS)
+#libebl_x86_64_pic_a_OBJECTS = $(am_libebl_x86_64_pic_a_OBJECTS)
 AM_V_P = $(am__v_P_$(V))
 am__v_P_ = $(am__v_P_$(AM_DEFAULT_VERBOSITY))
 am__v_P_0 = false
@@ -201,16 +201,15 @@
 am__v_CCLD_0 = @echo "  CCLD    " $@;
 am__v_CCLD_1 = 
 SOURCES = $(libebl_alpha_pic_a_SOURCES) $(libebl_arm_pic_a_SOURCES) \
-	$(libebl_i386_pic_a_SOURCES) $(libebl_ia64_pic_a_SOURCES) \
+	$(libebl_ia64_pic_a_SOURCES) \
 	$(libebl_ppc64_pic_a_SOURCES) $(libebl_ppc_pic_a_SOURCES) \
 	$(libebl_s390_pic_a_SOURCES) $(libebl_sh_pic_a_SOURCES) \
-	$(libebl_sparc_pic_a_SOURCES) $(libebl_x86_64_pic_a_SOURCES)
+	$(libebl_sparc_pic_a_SOURCES)
 DIST_SOURCES = $(libebl_alpha_pic_a_SOURCES) \
-	$(libebl_arm_pic_a_SOURCES) $(libebl_i386_pic_a_SOURCES) \
+	$(libebl_arm_pic_a_SOURCES)  \
 	$(libebl_ia64_pic_a_SOURCES) $(libebl_ppc64_pic_a_SOURCES) \
 	$(libebl_ppc_pic_a_SOURCES) $(libebl_s390_pic_a_SOURCES) \
-	$(libebl_sh_pic_a_SOURCES) $(libebl_sparc_pic_a_SOURCES) \
-	$(libebl_x86_64_pic_a_SOURCES)
+	$(libebl_sh_pic_a_SOURCES) $(libebl_sparc_pic_a_SOURCES)
 am__can_run_installinfo = \
   case $$AM_UPDATE_INFO_DIR in \
     n|no|NO) false;; \
@@ -278,7 +277,7 @@
 MAINT = #
 MAKEINFO = makeinfo
 MKDIR_P = /bin/mkdir -p
-MODVERSION = Build on R04220 2016-08-24T14:28:41+0800
+MODVERSION = Build on R04220 2016-08-24T10:35:18+0800
 MSGFMT = /usr/bin/msgfmt
 MSGFMT_015 = /usr/bin/msgfmt
 MSGMERGE = /usr/bin/msgmerge
@@ -372,8 +371,8 @@
 CLEANFILES = *.gcno *.gcda $(foreach m,$(modules), libebl_$(m).map \
 	libebl_$(m).so $(am_libebl_$(m)_pic_a_OBJECTS))
 textrel_check = if readelf -d $@ | fgrep -q TEXTREL; then exit 1; fi
-modules = i386 sh x86_64 ia64 alpha arm sparc ppc ppc64 s390
-libebl_pic = libebl_i386_pic.a libebl_sh_pic.a libebl_x86_64_pic.a \
+modules = sh ia64 alpha arm sparc ppc ppc64 s390
+libebl_pic = libebl_sh_pic.a \
 	     libebl_ia64_pic.a libebl_alpha_pic.a libebl_arm_pic.a \
 	     libebl_sparc_pic.a libebl_ppc_pic.a libebl_ppc64_pic.a \
 	     libebl_s390_pic.a
@@ -384,21 +383,21 @@
 #libelf = ../libelf/libelf.a
 libdw = ../libdw/libdw.so
 #libdw = ../libdw/libdw.a
-i386_SRCS = i386_init.c i386_symbol.c i386_corenote.c i386_cfi.c \
+#i386_SRCS = i386_init.c i386_symbol.c i386_corenote.c i386_cfi.c \
 	    i386_retval.c i386_regs.c i386_auxv.c i386_syscall.c
 
-cpu_i386 = ../libcpu/libcpu_i386.a
-libebl_i386_pic_a_SOURCES = $(i386_SRCS)
-am_libebl_i386_pic_a_OBJECTS = $(i386_SRCS:.c=.os)
+#cpu_i386 = ../libcpu/libcpu_i386.a
+#libebl_i386_pic_a_SOURCES = $(i386_SRCS)
+#am_libebl_i386_pic_a_OBJECTS = $(i386_SRCS:.c=.os)
 sh_SRCS = sh_init.c sh_symbol.c sh_corenote.c sh_regs.c sh_retval.c
 libebl_sh_pic_a_SOURCES = $(sh_SRCS)
 am_libebl_sh_pic_a_OBJECTS = $(sh_SRCS:.c=.os)
-x86_64_SRCS = x86_64_init.c x86_64_symbol.c x86_64_corenote.c x86_64_cfi.c \
+#x86_64_SRCS = x86_64_init.c x86_64_symbol.c x86_64_corenote.c x86_64_cfi.c \
 	      x86_64_retval.c x86_64_regs.c i386_auxv.c x86_64_syscall.c
 
-cpu_x86_64 = ../libcpu/libcpu_x86_64.a
-libebl_x86_64_pic_a_SOURCES = $(x86_64_SRCS)
-am_libebl_x86_64_pic_a_OBJECTS = $(x86_64_SRCS:.c=.os)
+#cpu_x86_64 = ../libcpu/libcpu_x86_64.a
+#libebl_x86_64_pic_a_SOURCES = $(x86_64_SRCS)
+#am_libebl_x86_64_pic_a_OBJECTS = $(x86_64_SRCS:.c=.os)
 ia64_SRCS = ia64_init.c ia64_symbol.c ia64_regs.c ia64_retval.c
 libebl_ia64_pic_a_SOURCES = $(ia64_SRCS)
 am_libebl_ia64_pic_a_OBJECTS = $(ia64_SRCS:.c=.os)
@@ -480,10 +479,10 @@
 	$(AM_V_AR)$(libebl_arm_pic_a_AR) libebl_arm_pic.a $(libebl_arm_pic_a_OBJECTS) $(libebl_arm_pic_a_LIBADD)
 	$(AM_V_at)$(RANLIB) libebl_arm_pic.a
 
-libebl_i386_pic.a: $(libebl_i386_pic_a_OBJECTS) $(libebl_i386_pic_a_DEPENDENCIES) $(EXTRA_libebl_i386_pic_a_DEPENDENCIES) 
-	$(AM_V_at)-rm -f libebl_i386_pic.a
-	$(AM_V_AR)$(libebl_i386_pic_a_AR) libebl_i386_pic.a $(libebl_i386_pic_a_OBJECTS) $(libebl_i386_pic_a_LIBADD)
-	$(AM_V_at)$(RANLIB) libebl_i386_pic.a
+#libebl_i386_pic.a: $(libebl_i386_pic_a_OBJECTS) $(libebl_i386_pic_a_DEPENDENCIES) $(EXTRA_libebl_i386_pic_a_DEPENDENCIES) 
+#	$(AM_V_at)-rm -f libebl_i386_pic.a
+#	$(AM_V_AR)$(libebl_i386_pic_a_AR) libebl_i386_pic.a $(libebl_i386_pic_a_OBJECTS) $(libebl_i386_pic_a_LIBADD)
+#	$(AM_V_at)$(RANLIB) libebl_i386_pic.a
 
 libebl_ia64_pic.a: $(libebl_ia64_pic_a_OBJECTS) $(libebl_ia64_pic_a_DEPENDENCIES) $(EXTRA_libebl_ia64_pic_a_DEPENDENCIES) 
 	$(AM_V_at)-rm -f libebl_ia64_pic.a
@@ -515,10 +514,10 @@
 	$(AM_V_AR)$(libebl_sparc_pic_a_AR) libebl_sparc_pic.a $(libebl_sparc_pic_a_OBJECTS) $(libebl_sparc_pic_a_LIBADD)
 	$(AM_V_at)$(RANLIB) libebl_sparc_pic.a
 
-libebl_x86_64_pic.a: $(libebl_x86_64_pic_a_OBJECTS) $(libebl_x86_64_pic_a_DEPENDENCIES) $(EXTRA_libebl_x86_64_pic_a_DEPENDENCIES) 
-	$(AM_V_at)-rm -f libebl_x86_64_pic.a
-	$(AM_V_AR)$(libebl_x86_64_pic_a_AR) libebl_x86_64_pic.a $(libebl_x86_64_pic_a_OBJECTS) $(libebl_x86_64_pic_a_LIBADD)
-	$(AM_V_at)$(RANLIB) libebl_x86_64_pic.a
+#libebl_x86_64_pic.a: $(libebl_x86_64_pic_a_OBJECTS) $(libebl_x86_64_pic_a_DEPENDENCIES) $(EXTRA_libebl_x86_64_pic_a_DEPENDENCIES) 
+#	$(AM_V_at)-rm -f libebl_x86_64_pic.a
+#	$(AM_V_AR)$(libebl_x86_64_pic_a_AR) libebl_x86_64_pic.a $(libebl_x86_64_pic_a_OBJECTS) $(libebl_x86_64_pic_a_LIBADD)
+#	$(AM_V_at)$(RANLIB) libebl_x86_64_pic.a
 
 mostlyclean-compile:
 	-rm -f *.$(OBJEXT)
@@ -539,14 +538,14 @@
 include ./$(DEPDIR)/arm_regs.Po
 include ./$(DEPDIR)/arm_retval.Po
 include ./$(DEPDIR)/arm_symbol.Po
-include ./$(DEPDIR)/i386_auxv.Po
-include ./$(DEPDIR)/i386_cfi.Po
-include ./$(DEPDIR)/i386_corenote.Po
-include ./$(DEPDIR)/i386_init.Po
-include ./$(DEPDIR)/i386_regs.Po
-include ./$(DEPDIR)/i386_retval.Po
-include ./$(DEPDIR)/i386_symbol.Po
-include ./$(DEPDIR)/i386_syscall.Po
+#include ./$(DEPDIR)/i386_auxv.Po
+#include ./$(DEPDIR)/i386_cfi.Po
+#include ./$(DEPDIR)/i386_corenote.Po
+#include ./$(DEPDIR)/i386_init.Po
+#include ./$(DEPDIR)/i386_regs.Po
+#include ./$(DEPDIR)/i386_retval.Po
+#include ./$(DEPDIR)/i386_symbol.Po
+#include ./$(DEPDIR)/i386_syscall.Po
 include ./$(DEPDIR)/ia64_init.Po
 include ./$(DEPDIR)/ia64_regs.Po
 include ./$(DEPDIR)/ia64_retval.Po
@@ -579,13 +578,6 @@
 include ./$(DEPDIR)/sparc_regs.Po
 include ./$(DEPDIR)/sparc_retval.Po
 include ./$(DEPDIR)/sparc_symbol.Po
-include ./$(DEPDIR)/x86_64_cfi.Po
-include ./$(DEPDIR)/x86_64_corenote.Po
-include ./$(DEPDIR)/x86_64_init.Po
-include ./$(DEPDIR)/x86_64_regs.Po
-include ./$(DEPDIR)/x86_64_retval.Po
-include ./$(DEPDIR)/x86_64_symbol.Po
-include ./$(DEPDIR)/x86_64_syscall.Po
 
 .c.o:
 	$(AM_V_CC)$(COMPILE) -MT $@ -MD -MP -MF $(DEPDIR)/$*.Tpo -c -o $@ $<
@@ -822,8 +814,8 @@
 		-Wl,-z,defs -Wl,--as-needed $(libelf) $(libdw) $(libmudflap)
 	$(textrel_check)
 
-libebl_i386.so: $(cpu_i386)
-libebl_x86_64.so: $(cpu_x86_64)
+#libebl_i386.so: $(cpu_i386)
+#libebl_x86_64.so: $(cpu_x86_64)
 
 install: install-am install-ebl-modules
 install-ebl-modules:
diff -uNr elfutils-0.150/Makefile elfutils-0.150.change_makefile/Makefile
--- elfutils-0.150/Makefile	2016-08-24 14:28:43.006363730 +0800
+++ elfutils-0.150.change_makefile/Makefile	2016-08-24 10:35:45.771545207 +0800
@@ -358,7 +358,7 @@
 pkginclude_HEADERS = version.h
 
 # Add doc back when we have some real content.
-SUBDIRS = config m4 lib libelf libebl libdwfl libdw libcpu libasm backends \
+SUBDIRS = config m4 lib libelf libebl libdwfl libdw libasm backends \
 	  src po tests
 
 EXTRA_DIST = elfutils.spec GPG-KEY NOTES EXCEPTION
diff -uNr elfutils-0.150/src/Makefile elfutils-0.150.change_makefile/src/Makefile
--- elfutils-0.150/src/Makefile	2016-08-24 14:28:43.134364130 +0800
+++ elfutils-0.150.change_makefile/src/Makefile	2016-08-24 10:50:27.408937610 +0800
@@ -352,12 +352,12 @@
 LEX_OUTPUT_ROOT = lex.yy
 LIBEBL_SUBDIR = elfutils
 LIBOBJS = 
-LIBS = 
+LIBS = -lz
 LTLIBOBJS = 
 MAINT = #
 MAKEINFO = makeinfo
 MKDIR_P = /bin/mkdir -p
 MODVERSION = Build on R04220 2016-08-24T10:35:18+0800
 MSGFMT = /usr/bin/msgfmt
 MSGFMT_015 = /usr/bin/msgfmt
 MSGMERGE = /usr/bin/msgmerge
diff -uNr elfutils-0.150/tests/Makefile elfutils-0.150.change_makefile/tests/Makefile
--- elfutils-0.150/tests/Makefile	2016-08-24 14:28:43.158364205 +0800
+++ elfutils-0.150.change_makefile/tests/Makefile	2016-08-24 10:51:48.828696796 +0800
@@ -678,12 +678,12 @@
 LEX_OUTPUT_ROOT = lex.yy
 LIBEBL_SUBDIR = elfutils
 LIBOBJS = 
-LIBS = 
+LIBS = -lz
 LTLIBOBJS = 
 MAINT = #
 MAKEINFO = makeinfo
 MKDIR_P = /bin/mkdir -p
 MODVERSION = Build on R04220 2016-08-24T14:28:41+0800
 MSGFMT = /usr/bin/msgfmt
 MSGFMT_015 = /usr/bin/msgfmt
 MSGMERGE = /usr/bin/msgmerge
```

### perf arm 交叉编译记录
* Makefile 添加外部库路径、外部头文件路径、依赖的库

patch 如下：
```patch
diff --git a/tools/perf/Makefile b/tools/perf/Makefile
index b0f164b..5e88839 100644
--- a/tools/perf/Makefile
+++ b/tools/perf/Makefile
@@ -30,11 +30,13 @@ include config/utilities.mak
 # Define LDFLAGS=-static to build a static binary.
 #
 # Define EXTRA_CFLAGS=-m64 or EXTRA_CFLAGS=-m32 as appropriate for cross-builds.
-#
+EXTRA_CFLAGS=-I/home/sunnogo/workshop/rgosm-build/public/include -I/home/sunnogo/workshop/prj/images/header -D_LITTLE_ENDIAN -L/home/sunnogo/workshop/prj/images/lib -L/home/sunnogo/workshop/prj/images/rootfs/lib
+
 # Define NO_DWARF if you do not want debug-info analysis feature at all.
 #
 # Define WERROR=0 to disable treating any warnings as errors.
-#
+WERROR=0
+
 # Define NO_NEWT if you do not want TUI support. (deprecated)
 #
 # Define NO_SLANG if you do not want TUI support.
@@ -111,7 +113,7 @@ ifdef NO_NEWT
 endif
 
 CFLAGS = -fno-omit-frame-pointer -ggdb3 -funwind-tables -Wall -Wextra -std=gnu99 $(CFLAGS_WERROR) $(CFLAGS_OPTIMIZE) $(EXTRA_WARNINGS) $(EXTRA_CFLAGS) $(PARSER_DEBUG_CFLAGS)
-EXTLIBS = -lpthread -lrt -lelf -lm
+EXTLIBS = -lpthread -lrt -lelf -lm -lebl -lz -ldl
 ALL_CFLAGS = $(CFLAGS) -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE
 ALL_LDFLAGS = $(LDFLAGS)
 STRIP ?= strip
```
