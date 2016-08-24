Linux Kernel tools perf
=======================
以下译自 [en.wikipedia](https://en.wikipedia.org/wiki/Perf_(Linux))
> **perf** （有时亦称为 "Perf Events" 或 perf 工具，原始名称为 "Performance Counters for linux", PCL），是一个 Linux 性能分析工具，从 Linux 内核版本 2.6.31 导入。用户空间控制工具名为 perf，在命令行使用，提供一系列子命令，适用于整个系统（包含用户空间、内核空间代码）的 profiling 数据统计（即性能剖析数据统计）。
> 
> perf 支持硬件性能计数器、tracepoints、软件性能计数器（如 hrtimer），和动态 probes （比如 kprobes 或uprobes）。IBM 在 2012 年将 perf 与 Oprofile 评为两个最常用的 linux performance counter profiling tools。

MIPS
----
截止

ARM
---
依赖 zlib、elfutils。

交叉编译记录参考，丫凡的博文《[交叉编译 perf for arm](http://freenix.blogcn.com/articles/%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-perf-for-arm.html)》。

* 内核版本：3.10.18
* arm 交叉编译工具链：arm-cortex_a9-linux-gnueabi-gcc

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



### perf arm 交叉编译记录
*
