title: kconfig-frontends 简介
date: 2018-11-01 09:14:22
toc: true
tags: [kconfig]
categories: programmer
keywords: [kconfig, standalone, conf, menuconfig, defconfig, alldefconfig, configuration]
description: Kconfig-frontends - “官方” standalone kconfig 工具，使用简介。
---

Kconfig + Kbuild 为内核编译的配置和编译框架。其中 Kconfig 作为一个典型的配置框架，被很多开发人员单独使用，典型的项目有 BuildRoot、uboot。

也有一些人将 Kconfig 的源码从内核中剥出，作为一个独立的组件。比如:

* [Kconfiglib](https://github.com/ulfalizer/Kconfiglib), A flexible Python 2/3 Kconfig implementation and library
* [guillon/kconfig](https://github.com/guillon/kconfig), 通过 kconfig/config.sh 脚本，实现 kconfig 编译和目标运行
* [kconfig-frontends](http://ymorin.is-a-geek.org/projects/kconfig-frontends)，与内核一起发布，有点官方 standalone kconfig 的意思。需要编译、安装，之后通过 kconfig-conf 等命令运行。

这里介绍 Kconfig-frontend 从零到使用的过程。

## 下载编译

1. wget http://ymorin.is-a-geek.org/download/kconfig-frontends/kconfig-frontends-3.19.0.0.tar.xz
2. tar xvf kconfig-frontends-3.19.0.0.tar.xz 
3. sudo apt-get install gperf
4. ./configure --enable-conf --enable-mconf --disable-shared --enable-static
5. make
6. make install

## 使用记录

### 类 make alldefconfig

* `kconfig-conf --alldefconfig Kconfig`，其中文件 Kconfig 为配置入口。

```
jeromesun@kmcb0220:~/workshop/hello$ kconfig-conf --alldefconfig Kconfig 
#
# configuration written to .config
#
jeromesun@kmcb0220:~/workshop/hello$ ls
configs  hello.c  hello.d  hello.o  Kconfig  lib  Makefile
jeromesun@kmcb0220:~/workshop/hello$ vi .config 
  1 #
  2 # Automatically generated file; DO NOT EDIT.
  3 # MYAPP Configuration
  4 #
  5 CONFIG_CROSS_COMPILE=""
  6 CONFIG_ENABLE_LOGGING=y
  7 CONFIG_DEFAULT_LOGLEVEL=3
  8 CONFIG_LOGGING_TIME=y
  9 # CONFIG_DEBUG_LIST is not set
 10 # CONFIG_FOO is not set
```

### 类 make menuconfig

* `kconfig-mconf Kconfig`

```
jeromesun@kmcb0220:~/workshop/hello$ kconfig-mconf Kconfig
 .config - MYAPP Configuration
 ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  ┌───────────────────────────────────────────── MYAPP Configuration ─────────────────────────────────────────────┐
  │  Arrow keys navigate the menu.  <Enter> selects submenus ---> (or empty submenus ----).  Highlighted letters  │  
  │  are hotkeys.  Pressing <Y> includes, <N> excludes, <M> modularizes features.  Press <Esc><Esc> to exit, <?>  │  
  │  for Help, </> for Search.  Legend: [*] built-in  [ ] excluded  <M> module  < > module capable                │  
  │                                                                                                               │  
  │ ┌───────────────────────────────────────────────────────────────────────────────────────────────────────────┐ │  
  │ │                  ()  Cross-compiler tool prefix                                                           │ │  
  │ │                  [*] Enable log functions                                                                 │ │  
  │ │                  (3)   Default message log level (1-4)                                                    │ │  
  │ │                  [*]   Show timing information on log functions                                           │ │  
  │ │                  [ ] Debug linked list manipulation                                                       │ │  
  │ │                  [ ] Foo                                                                                  │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ │                                                                                                           │ │  
  │ └───────────────────────────────────────────────────────────────────────────────────────────────────────────┘ │  
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────┤  
  │                           <Select>    < Exit >    < Help >    < Save >    < Load >                            │  
  └───────────────────────────────────────────────────────────────────────────────────────────────────────────────┘  
                                                                                                                     
configuration written to .config

*** End of the configuration.
*** Execute 'make' to start the build or try 'make help'.

jeromesun@kmcb0220:~/workshop/hello$ cat .config
#
# Automatically generated file; DO NOT EDIT.
# MYAPP Configuration
#
CONFIG_CROSS_COMPILE=""
CONFIG_ENABLE_LOGGING=y
CONFIG_DEFAULT_LOGLEVEL=3
CONFIG_LOGGING_TIME=y
# CONFIG_DEBUG_LIST is not set
CONFIG_FOO=y
CONFIG_BAR=y
```

## 编译 log

### 下载、解压

```
jeromesun@km:~/workshop$ wget http://ymorin.is-a-geek.org/download/kconfig-frontends/kconfig-frontends-3.19.0.0.tar.xz
--2018-10-26 23:21:48--  http://ymorin.is-a-geek.org/download/kconfig-frontends/kconfig-frontends-3.19.0.0.tar.xz
Resolving ymorin.is-a-geek.org (ymorin.is-a-geek.org)... 90.76.108.19
Connecting to ymorin.is-a-geek.org (ymorin.is-a-geek.org)|90.76.108.19|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 370660 (362K) [application/octet-stream]
Saving to: ‘kconfig-frontends-3.19.0.0.tar.xz’

kconfig-frontends-3.19.0.0.tar.xz  100%[===============================================================>] 361.97K  14.2KB/s    in 17s     

2018-10-26 23:22:06 (21.5 KB/s) - ‘kconfig-frontends-3.19.0.0.tar.xz’ saved [370660/370660]

jeromesun@km:~/workshop$ 
jeromesun@km:~/workshop$ tar xvf kconfig-frontends-3.19.0.0.tar.xz 
kconfig-frontends-3.19.0.0/
kconfig-frontends-3.19.0.0/frontends/
kconfig-frontends-3.19.0.0/frontends/qconf/
kconfig-frontends-3.19.0.0/frontends/qconf/Makefile.in
kconfig-frontends-3.19.0.0/frontends/qconf/qconf.h
kconfig-frontends-3.19.0.0/frontends/qconf/qconf.cc.patch
kconfig-frontends-3.19.0.0/frontends/qconf/qconf.cc
kconfig-frontends-3.19.0.0/frontends/qconf/Makefile.am
kconfig-frontends-3.19.0.0/frontends/conf/
kconfig-frontends-3.19.0.0/frontends/conf/Makefile.in
kconfig-frontends-3.19.0.0/frontends/conf/Makefile.am
kconfig-frontends-3.19.0.0/frontends/conf/conf.c
kconfig-frontends-3.19.0.0/frontends/Makefile.in
kconfig-frontends-3.19.0.0/frontends/gconf/
kconfig-frontends-3.19.0.0/frontends/gconf/Makefile.in
kconfig-frontends-3.19.0.0/frontends/gconf/gconf.glade
kconfig-frontends-3.19.0.0/frontends/gconf/gconf.c
kconfig-frontends-3.19.0.0/frontends/gconf/gconf.c.patch
kconfig-frontends-3.19.0.0/frontends/gconf/Makefile.am
kconfig-frontends-3.19.0.0/frontends/mconf/
kconfig-frontends-3.19.0.0/frontends/mconf/Makefile.in
kconfig-frontends-3.19.0.0/frontends/mconf/mconf.c
kconfig-frontends-3.19.0.0/frontends/mconf/Makefile.am
kconfig-frontends-3.19.0.0/frontends/Makefile.am
kconfig-frontends-3.19.0.0/frontends/nconf/
kconfig-frontends-3.19.0.0/frontends/nconf/nconf.h
kconfig-frontends-3.19.0.0/frontends/nconf/Makefile.in
kconfig-frontends-3.19.0.0/frontends/nconf/nconf.c
kconfig-frontends-3.19.0.0/frontends/nconf/Makefile.am
kconfig-frontends-3.19.0.0/frontends/nconf/nconf.gui.c
kconfig-frontends-3.19.0.0/INSTALL
kconfig-frontends-3.19.0.0/utils/
kconfig-frontends-3.19.0.0/utils/tweak.in
kconfig-frontends-3.19.0.0/utils/Makefile.in
kconfig-frontends-3.19.0.0/utils/merge
kconfig-frontends-3.19.0.0/utils/gettext.c
kconfig-frontends-3.19.0.0/utils/diff
kconfig-frontends-3.19.0.0/utils/Makefile.am
kconfig-frontends-3.19.0.0/utils/tweak.in.patch
kconfig-frontends-3.19.0.0/README
kconfig-frontends-3.19.0.0/Makefile.in
kconfig-frontends-3.19.0.0/aclocal.m4
kconfig-frontends-3.19.0.0/COPYING
kconfig-frontends-3.19.0.0/AUTHORS
kconfig-frontends-3.19.0.0/libs/
kconfig-frontends-3.19.0.0/libs/images/
kconfig-frontends-3.19.0.0/libs/images/Makefile.in
kconfig-frontends-3.19.0.0/libs/images/Makefile.am
kconfig-frontends-3.19.0.0/libs/images/images.c_orig
kconfig-frontends-3.19.0.0/libs/Makefile.in
kconfig-frontends-3.19.0.0/libs/lxdialog/
kconfig-frontends-3.19.0.0/libs/lxdialog/util.c
kconfig-frontends-3.19.0.0/libs/lxdialog/checklist.c
kconfig-frontends-3.19.0.0/libs/lxdialog/yesno.c
kconfig-frontends-3.19.0.0/libs/lxdialog/Makefile.in
kconfig-frontends-3.19.0.0/libs/lxdialog/menubox.c
kconfig-frontends-3.19.0.0/libs/lxdialog/dialog.h
kconfig-frontends-3.19.0.0/libs/lxdialog/Makefile.am
kconfig-frontends-3.19.0.0/libs/lxdialog/textbox.c
kconfig-frontends-3.19.0.0/libs/lxdialog/inputbox.c
kconfig-frontends-3.19.0.0/libs/Makefile.am
kconfig-frontends-3.19.0.0/libs/parser/
kconfig-frontends-3.19.0.0/libs/parser/util.c
kconfig-frontends-3.19.0.0/libs/parser/yconf.c
kconfig-frontends-3.19.0.0/libs/parser/yconf.y.patch
kconfig-frontends-3.19.0.0/libs/parser/lkc_proto.h
kconfig-frontends-3.19.0.0/libs/parser/confdata.c
kconfig-frontends-3.19.0.0/libs/parser/symbol.c
kconfig-frontends-3.19.0.0/libs/parser/Makefile.in
kconfig-frontends-3.19.0.0/libs/parser/lkc.h
kconfig-frontends-3.19.0.0/libs/parser/yconf.y
kconfig-frontends-3.19.0.0/libs/parser/list.h
kconfig-frontends-3.19.0.0/libs/parser/expr.c
kconfig-frontends-3.19.0.0/libs/parser/menu.c
kconfig-frontends-3.19.0.0/libs/parser/expr.h
kconfig-frontends-3.19.0.0/libs/parser/hconf.gperf
kconfig-frontends-3.19.0.0/libs/parser/lconf.l
kconfig-frontends-3.19.0.0/libs/parser/Makefile.am
kconfig-frontends-3.19.0.0/libs/parser/lconf.c
kconfig-frontends-3.19.0.0/scripts/
kconfig-frontends-3.19.0.0/scripts/version.sh
kconfig-frontends-3.19.0.0/scripts/Makefile.in
kconfig-frontends-3.19.0.0/scripts/ksync.list
kconfig-frontends-3.19.0.0/scripts/ksync.sh
kconfig-frontends-3.19.0.0/scripts/Makefile.am
kconfig-frontends-3.19.0.0/scripts/.autostuff/
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/missing
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/depcomp
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/config.sub
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/ar-lib
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/ltmain.sh
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/compile
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/config.guess
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/ylwrap
kconfig-frontends-3.19.0.0/scripts/.autostuff/scripts/install-sh
kconfig-frontends-3.19.0.0/scripts/.autostuff/m4/
kconfig-frontends-3.19.0.0/scripts/.autostuff/m4/lt~obsolete.m4
kconfig-frontends-3.19.0.0/scripts/.autostuff/m4/ltoptions.m4
kconfig-frontends-3.19.0.0/scripts/.autostuff/m4/libtool.m4
kconfig-frontends-3.19.0.0/scripts/.autostuff/m4/ltversion.m4
kconfig-frontends-3.19.0.0/scripts/.autostuff/m4/ltsugar.m4
kconfig-frontends-3.19.0.0/scripts/.autostuff/config.h.in
kconfig-frontends-3.19.0.0/bootstrap
kconfig-frontends-3.19.0.0/.version
kconfig-frontends-3.19.0.0/Makefile.am
kconfig-frontends-3.19.0.0/configure
kconfig-frontends-3.19.0.0/configure.ac
kconfig-frontends-3.19.0.0/docs/
kconfig-frontends-3.19.0.0/docs/kconfig-language.txt
kconfig-frontends-3.19.0.0/docs/kconfig.txt
kconfig-frontends-3.19.0.0/docs/Makefile.in
kconfig-frontends-3.19.0.0/docs/Makefile.am
jeromesun@km:~/workshop$ cd kconfig-frontends-3.19.0.0/
```

### 配置

如果没安装 gperf 会提示 `configure: error: can not find gperf`。

```
jeromesun@km:~/workshop/kconfig-frontends-3.19.0.0$ ./configure --help
`configure' configures kconfig-frontends 3.19.0.0 to adapt to many kinds of systems.

Usage: ./configure [OPTION]... [VAR=VALUE]...

To assign environment variables (e.g., CC, CFLAGS...), specify them as
VAR=VALUE.  See below for descriptions of some of the useful variables.

Defaults for the options are specified in brackets.

Configuration:
  -h, --help              display this help and exit
      --help=short        display options specific to this package
      --help=recursive    display the short help of all the included packages
  -V, --version           display version information and exit
  -q, --quiet, --silent   do not print `checking ...' messages
      --cache-file=FILE   cache test results in FILE [disabled]
  -C, --config-cache      alias for `--cache-file=config.cache'
  -n, --no-create         do not create output files
      --srcdir=DIR        find the sources in DIR [configure dir or `..']

Installation directories:
  --prefix=PREFIX         install architecture-independent files in PREFIX
                          [/usr/local]
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                          [PREFIX]

By default, `make install' will install all the files in
`/usr/local/bin', `/usr/local/lib' etc.  You can specify
an installation prefix other than `/usr/local' using `--prefix',
for instance `--prefix=$HOME'.

For better control, use the options below.

Fine tuning of the installation directories:
  --bindir=DIR            user executables [EPREFIX/bin]
  --sbindir=DIR           system admin executables [EPREFIX/sbin]
  --libexecdir=DIR        program executables [EPREFIX/libexec]
  --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
  --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
  --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
  --runstatedir=DIR       modifiable per-process data [LOCALSTATEDIR/run]
  --libdir=DIR            object code libraries [EPREFIX/lib]
  --includedir=DIR        C header files [PREFIX/include]
  --oldincludedir=DIR     C header files for non-gcc [/usr/include]
  --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
  --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
  --infodir=DIR           info documentation [DATAROOTDIR/info]
  --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
  --mandir=DIR            man documentation [DATAROOTDIR/man]
  --docdir=DIR            documentation root
                          [DATAROOTDIR/doc/kconfig-frontends]
  --htmldir=DIR           html documentation [DOCDIR]
  --dvidir=DIR            dvi documentation [DOCDIR]
  --pdfdir=DIR            pdf documentation [DOCDIR]
  --psdir=DIR             ps documentation [DOCDIR]

Program names:
  --program-prefix=PREFIX            prepend PREFIX to installed program names
  --program-suffix=SUFFIX            append SUFFIX to installed program names
  --program-transform-name=PROGRAM   run sed PROGRAM on installed program names

System types:
  --build=BUILD     configure for building on BUILD [guessed]
  --host=HOST       cross-compile to build programs to run on HOST [BUILD]

Optional Features:
  --disable-option-checking  ignore unrecognized --enable/--with options
  --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)
  --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]
  --enable-silent-rules   less verbose build output (undo: "make V=1")
  --disable-silent-rules  verbose build output (undo: "make V=0")
  --enable-dependency-tracking
                          do not reject slow dependency extractors
  --disable-dependency-tracking
                          speeds up one-time build
  --enable-static[=PKGS]  build static libraries [default=no]
  --enable-shared[=PKGS]  build shared libraries [default=yes]
  --enable-fast-install[=PKGS]
                          optimize for fast installation [default=yes]
  --disable-libtool-lock  avoid locking (might break parallel builds)
  --disable-wall          build with -Wall (default=yes)
  --enable-werror         build with -Werror (default=no)
  --enable-root-menu-prompt=PROMPT
                          set the root-menu prompt (default=Configuration)
  --enable-config-prefix=PREFIX
                          the prefix to the config option (default=CONFIG_)
  --disable-utils         install utilities to manage .config files
                          (default=yes)
  --disable-L10n          enable localisation (L10n) (default=auto)
  --disable-conf          conf, the stdin-based frontend (default=auto)
  --disable-mconf         mconf, the traditional ncurses-based frontend
                          (default=auto)
  --disable-nconf         nconf, the modern ncurses-based frontend
                          (default=auto)
  --disable-gconf         gconf, the GTK-based frontend (default=auto)
  --disable-qconf         qconf, the QT-based frontend (default=auto)
  --enable-frontends=list enables only the set of frontends in comma-separated
                          'list' (default: auto selection), takes precedence
                          over all --enable-*conf, above

Optional Packages:
  --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]
  --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)
  --with-pic[=PKGS]       try to use only PIC/non-PIC objects [default=use
                          both]
  --with-aix-soname=aix|svr4|both
                          shared library versioning (aka "SONAME") variant to
                          provide on AIX, [default=aix].
  --with-gnu-ld           assume the C compiler uses GNU ld [default=no]
  --with-sysroot[=DIR]    Search for dependent libraries within DIR (or the
                          compiler's sysroot if not specified).

Some influential environment variables:
  CC          C compiler command
  CFLAGS      C compiler flags
  LDFLAGS     linker flags, e.g. -L<lib dir> if you have libraries in a
              nonstandard directory <lib dir>
  LIBS        libraries to pass to the linker, e.g. -l<library>
  CPPFLAGS    (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if
              you have headers in a nonstandard directory <include dir>
  LT_SYS_LIBRARY_PATH
              User-defined run-time library search path.
  CPP         C preprocessor
  CXX         C++ compiler command
  CXXFLAGS    C++ compiler flags
  CXXCPP      C++ preprocessor
  PKG_CONFIG  path to pkg-config utility
  PKG_CONFIG_PATH
              directories to add to pkg-config's search path
  PKG_CONFIG_LIBDIR
              path overriding pkg-config's built-in search path
  YACC        The `Yet Another Compiler Compiler' implementation to use.
              Defaults to the first program found out of: `bison -y', `byacc',
              `yacc'.
  YFLAGS      The list of arguments that will be passed by default to $YACC.
              This script will default YFLAGS to the empty string to avoid a
              default value of `-d' given by some make applications.
  gtk_CFLAGS  C compiler flags for gtk, overriding pkg-config
  gtk_LIBS    linker flags for gtk, overriding pkg-config
  qt4_CFLAGS  C compiler flags for qt4, overriding pkg-config
  qt4_LIBS    linker flags for qt4, overriding pkg-config
  MOC         Qt meta object compiler (moc) command
  conf_EXTRA_LIBS
              Extra libraries to build the conf frontend
  gconf_EXTRA_LIBS
              Extra libraries to build the gconf frontend
  mconf_EXTRA_LIBS
              Extra libraries to build the mconf frontend
  nconf_EXTRA_LIBS
              Extra libraries to build the nconf frontend
  qconf_EXTRA_LIBS
              Extra libraries to build the qconf frontend

Use these variables to override the choices made by `configure' or to help
it to find libraries and programs with nonstandard names/locations.

Report bugs to <yann.morin.1998@free.fr>.
jeromesun@km:~/workshop/kconfig-frontends-3.19.0.0$ ./configure --enable-conf --enable-mconf --disable-shared --enable-static
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... no
checking for mawk... mawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether make supports nested variables... (cached) yes
checking for style of include used by make... GNU
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc understands -c and -o together... yes
checking dependency style of gcc... gcc3
checking for ar... ar
checking the archiver (ar) interface... ar
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking how to print strings... printf
checking for a sed that does not truncate output... /bin/sed
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for fgrep... /bin/grep -F
checking for ld used by gcc... /usr/bin/ld
checking if the linker (/usr/bin/ld) is GNU ld... yes
checking for BSD- or MS-compatible name lister (nm)... /usr/bin/nm -B
checking the name lister (/usr/bin/nm -B) interface... BSD nm
checking whether ln -s works... yes
checking the maximum length of command line arguments... 1572864
checking how to convert x86_64-pc-linux-gnu file names to x86_64-pc-linux-gnu format... func_convert_file_noop
checking how to convert x86_64-pc-linux-gnu file names to toolchain format... func_convert_file_noop
checking for /usr/bin/ld option to reload object files... -r
checking for objdump... objdump
checking how to recognize dependent libraries... pass_all
checking for dlltool... no
checking how to associate runtime and link libraries... printf %s\n
checking for archiver @FILE support... @
checking for strip... strip
checking for ranlib... ranlib
checking command to parse /usr/bin/nm -B output from gcc object... ok
checking for sysroot... no
checking for a working dd... /bin/dd
checking how to truncate binary pipes... /bin/dd bs=4096 count=1
checking for mt... mt
checking if mt is a manifest tool... no
checking how to run the C preprocessor... gcc -E
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking for dlfcn.h... yes
checking for objdir... .libs
checking if gcc supports -fno-rtti -fno-exceptions... no
checking for gcc option to produce PIC... -fPIC -DPIC
checking if gcc PIC flag -fPIC -DPIC works... yes
checking if gcc static flag -static works... yes
checking if gcc supports -c -o file.o... yes
checking if gcc supports -c -o file.o... (cached) yes
checking whether the gcc linker (/usr/bin/ld -m elf_x86_64) supports shared libraries... yes
checking dynamic linker characteristics... GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking whether stripping libraries is possible... yes
checking if libtool supports shared libraries... yes
checking whether to build shared libraries... no
checking whether to build static libraries... yes
checking for gcc... (cached) gcc
checking whether we are using the GNU C compiler... (cached) yes
checking whether gcc accepts -g... (cached) yes
checking for gcc option to accept ISO C89... (cached) none needed
checking whether gcc understands -c and -o together... (cached) yes
checking dependency style of gcc... (cached) gcc3
checking for g++... g++
checking whether we are using the GNU C++ compiler... yes
checking whether g++ accepts -g... yes
checking dependency style of g++... gcc3
checking how to run the C++ preprocessor... g++ -E
checking for ld used by g++... /usr/bin/ld -m elf_x86_64
checking if the linker (/usr/bin/ld -m elf_x86_64) is GNU ld... yes
checking whether the g++ linker (/usr/bin/ld -m elf_x86_64) supports shared libraries... yes
checking for g++ option to produce PIC... -fPIC -DPIC
checking if g++ PIC flag -fPIC -DPIC works... yes
checking if g++ static flag -static works... yes
checking if g++ supports -c -o file.o... yes
checking if g++ supports -c -o file.o... (cached) yes
checking whether the g++ linker (/usr/bin/ld -m elf_x86_64) supports shared libraries... yes
checking dynamic linker characteristics... (cached) GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking for inline... inline
checking whether make sets $(MAKE)... (cached) yes
checking for gperf... gperf
checking for pkg-config... /usr/bin/pkg-config
checking pkg-config is at least version 0.9.0... yes
checking for flex... flex
checking lex output file root... lex.yy
checking lex library... -lfl
checking whether yytext is a pointer... yes
checking for bison... bison -y
checking libintl.h usability... yes
checking libintl.h presence... yes
checking for libintl.h... yes
checking whether gettext is declared... yes
checking for library containing gettext... none required
checking ncursesw/curses.h usability... yes
checking ncursesw/curses.h presence... yes
checking for ncursesw/curses.h... yes
checking for library containing setupterm... -ltinfo
checking for library containing initscr... -lncursesw
checking for library containing new_panel... -lpanelw
checking for library containing menu_init... -lmenuw
checking for gtk... no
checking for qt4... no
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating docs/Makefile
config.status: creating libs/Makefile
config.status: creating libs/images/Makefile
config.status: creating libs/lxdialog/Makefile
config.status: creating libs/parser/Makefile
config.status: creating frontends/Makefile
config.status: creating frontends/conf/Makefile
config.status: creating frontends/mconf/Makefile
config.status: creating frontends/nconf/Makefile
config.status: creating frontends/gconf/Makefile
config.status: creating frontends/qconf/Makefile
config.status: creating utils/Makefile
config.status: creating scripts/Makefile
config.status: creating scripts/.autostuff/config.h
config.status: executing depfiles commands
config.status: executing libtool commands
configure: 
configure: Configured with:
configure: - parser library     : static
configure:   - root-menu prompt : Configuration
configure:   - config prefix    : CONFIG_
configure: - frontends          : conf mconf nconf
configure:   - transform name   : s&^&kconfig-&
configure:   - localised        : yes
configure: - install utilities  : yes
configure: - CFLAGS CXXFLAGS    : -Wall 
```

### 编译、安装

```
jeromesun@km:~/workshop/kconfig-frontends-3.19.0.0$ make
Making all in docs
Making all in libs
Making all in parser
  GPERF  hconf.c
  CC       libkconfig_parser_la-yconf.lo
  CCLD     libkconfig-parser.la
ar: `u' modifier ignored since `D' is the default (see `U')
Making all in lxdialog
  CC       libkconfig_lxdialog_a-checklist.o
  CC       libkconfig_lxdialog_a-inputbox.o
  CC       libkconfig_lxdialog_a-menubox.o
  CC       libkconfig_lxdialog_a-textbox.o
  CC       libkconfig_lxdialog_a-util.o
  CC       libkconfig_lxdialog_a-yesno.o
  AR       libkconfig-lxdialog.a
ar: `u' modifier ignored since `D' is the default (see `U')
Making all in frontends
Making all in conf
  CC       conf-conf.o
  CCLD     conf
Making all in mconf
  CC       mconf-mconf.o
  CCLD     mconf
Making all in nconf
  CC       nconf-nconf.o
  CC       nconf-nconf.gui.o
  CCLD     nconf
Making all in scripts
Making all in utils
  CC       gettext-gettext.o
  CCLD     gettext
  GEN      tweak
jeromesun@km:~/workshop/kconfig-frontends-3.19.0.0$ sudo make install
Making install in docs
 /bin/mkdir -p '/usr/local/share/doc/kconfig-frontends'
 /usr/bin/install -c -m 644 kconfig-language.txt kconfig.txt '/usr/local/share/doc/kconfig-frontends'
Making install in libs
Making install in parser
 /bin/mkdir -p '/usr/local/lib'
 /bin/bash ../../libtool   --mode=install /usr/bin/install -c   libkconfig-parser.la '/usr/local/lib'
libtool: install: /usr/bin/install -c .libs/libkconfig-parser.lai /usr/local/lib/libkconfig-parser.la
libtool: install: /usr/bin/install -c .libs/libkconfig-parser.a /usr/local/lib/libkconfig-parser.a
libtool: install: chmod 644 /usr/local/lib/libkconfig-parser.a
libtool: install: ranlib /usr/local/lib/libkconfig-parser.a
libtool: finish: PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/sbin" ldconfig -n /usr/local/lib
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the '-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the 'LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the 'LD_RUN_PATH' environment variable
     during linking
   - use the '-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to '/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
 /bin/mkdir -p '/usr/local/include/kconfig'
 /usr/bin/install -c -m 644 list.h lkc.h expr.h lkc_proto.h '/usr/local/include/kconfig'
Making install in lxdialog
Making install in frontends
Making install in conf
 /bin/mkdir -p '/usr/local/bin'
  /bin/bash ../../libtool   --mode=install /usr/bin/install -c conf '/usr/local/bin/./kconfig-conf'
libtool: install: /usr/bin/install -c conf /usr/local/bin/./kconfig-conf
Making install in mconf
 /bin/mkdir -p '/usr/local/bin'
  /bin/bash ../../libtool   --mode=install /usr/bin/install -c mconf '/usr/local/bin/./kconfig-mconf'
libtool: install: /usr/bin/install -c mconf /usr/local/bin/./kconfig-mconf
Making install in nconf
 /bin/mkdir -p '/usr/local/bin'
  /bin/bash ../../libtool   --mode=install /usr/bin/install -c nconf '/usr/local/bin/./kconfig-nconf'
libtool: install: /usr/bin/install -c nconf /usr/local/bin/./kconfig-nconf
Making install in scripts
Making install in utils
 /bin/mkdir -p '/usr/local/bin'
  /bin/bash ../libtool   --mode=install /usr/bin/install -c gettext '/usr/local/bin/./kconfig-gettext'
libtool: install: /usr/bin/install -c gettext /usr/local/bin/./kconfig-gettext
 /bin/mkdir -p '/usr/local/bin'
 /usr/bin/install -c tweak '/usr/local/bin/./kconfig-tweak'
 /bin/mkdir -p '/usr/local/bin'
 /usr/bin/install -c diff '/usr/local/bin/./kconfig-diff'
 /usr/bin/install -c merge '/usr/local/bin/./kconfig-merge'
jeromesun@km:~/workshop/kconfig-frontends-3.19.0.0$ 
```
