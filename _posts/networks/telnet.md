title: Linux telnet 服务器调研
date: 2018-10-26 16:03:55
toc: true
tags: [networks, telnet]
categories: networks
keywords: [telnet, telnetd, telnet 服务器, server, inetd, xinetd, openbsd-inetd, tcpd, tcp-wrapper, utelnetd, ]
description: Linux telnet 服务器调研、验证与分析。
---

## linux telnetd 分析

Linux 发行版本视 telnet 为不安全工具，对 telnet 的支持比较差。
目前发行版主要使用 inetd (internet “super-server”) + telnetd 的形式，并没有独立的 telnet 服务器。一般只有在嵌入式系统中才有 telnet 服务器。

* [telnetd](https://packages.debian.org/jessie/telnetd), 源码隶属 netkit-telnet 
  + xinetd + telnetd
  + openbsd-inetd + tcpd + telnetd
* standalone telnetd:
  + utelnetd, https://wiki.gentoo.org/wiki/Utelnetd, 代码https://public.pengutronix.de/software/utelnetd/, 用于 optware/gentoo 等。
  + busybox telnetd

优缺点分析：
* inetd + telnetd
  + 目前各大发行版本支持
  + 正常情况下，可直接安装使用。目前在 ubuntu 上验证，即装即可用，但是在 debian jessie 上，不可用。下文详细介绍 fix 过程，fix 后 openbsd-inetd (debian 默认 inetd) + telnetd 可用。
  + 不方便维护，除 telnetd，还需要引进 inetd，多一个故障点
  + 多一个 inetd，多一个代码维护点
* standalone telnetd
  + 代码少，以 utelnetd 来讲，空行算进去，总的也才 600 多行。
  + 相比 inetd + telnetd 方式，部署简单，方便维护。
  + busybox 的 telnetd 在整个 busybox 框架下，如果单独剥离出来，需要一定工作量。相比之下，utelnetd 很方便。


### 关于 inetd

网上推荐使用 xinetd，redhat 系用 xinetd 比较多。debian 系支持的比较少，像 update-inetd 就明确安装时就说明不完全支持 xinetd。

>      inetd listens for connections on certain internet sockets.  When a connection is found on one of its sockets, it decides what service the socket corresponds to, and invokes a program to service the request.  After the program is finished, it continues to listen on the socket (except in some cases which will be described below).  Essentially, inetd allows running one daemon to invoke several others, reducing load on the system.

## utelnetd 验证

验证可用。

验证 log:

```
a@xxx:~$ sudo lsof -i:23 -n
a@xxx:~$ sudo utelnetd -d
telnetd: starting
  port: 23; interface: any; login program: /bin/login
a@xxx:~$ 
a@xxx:~$ sudo lsof -i:23 -n
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
utelnetd 27898 root    3u  IPv4 120819      0t0  TCP *:telnet (LISTEN)
a@xxx:~$ 
a@xxx:~$ telnet localhost 
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Debian GNU/Linux 8
xxx login: a
Password: 
Last login: Fri Oct 26 22:46:25 CST 2018 on pts/11
Linux xxx 3.16.0-5-amd64 #1 SMP Debian 3.16.51-3+deb8u1 (2018-01-08) x86_64

Unauthorized access and/or use are prohibited.
All access and/or use are subject to monitoring.

a@xxx:~$ ls
libperl4-corelibs-perl_0.003-1_all.deb  tcpd                       utelnetd
lsof_4.86+dfsg-1_amd64.deb              telnetd_0.17-41_amd64.deb
a@xxx:~$ logout
Connection closed by foreign host.
```

### 编译记录

```
jeromesun@km:~/workshop$ tar xvf utelnetd-0.1.11.tar.gz 
utelnetd-0.1.11/
utelnetd-0.1.11/utelnetd.c
utelnetd-0.1.11/LICENSE
utelnetd-0.1.11/Makefile
utelnetd-0.1.11/ChangeLog
utelnetd-0.1.11/README
jeromesun@km:~/workshop$ cd utelnetd-0.1.11/
jeromesun@km:~/workshop/utelnetd-0.1.11$ make
gcc -I. -pipe -DSHELLPATH=\"/bin/login\" -Wall -fomit-frame-pointer   -c -o utelnetd.o utelnetd.c
utelnetd.c: In function ‘perror_msg_and_die’:
utelnetd.c:160:2: warning: format not a string literal and no format arguments [-Wformat-security]
  fprintf(stderr,text);
  ^
utelnetd.c: In function ‘make_new_session’:
utelnetd.c:264:10: warning: variable ‘t2’ set but not used [-Wunused-but-set-variable]
  int t1, t2;
          ^
utelnetd.c:264:6: warning: variable ‘t1’ set but not used [-Wunused-but-set-variable]
  int t1, t2;
      ^
utelnetd.c: In function ‘main’:
utelnetd.c:561:7: warning: pointer targets in passing argument 3 of ‘accept’ differ in signedness [-Wpointer-sign]
       &salen)) < 0) {
       ^
In file included from utelnetd.c:45:0:
/usr/include/x86_64-linux-gnu/sys/socket.h:243:12: note: expected ‘socklen_t * restrict {aka unsigned int * restrict}’ but argument is of type ‘int *’
 extern int accept (int __fd, __SOCKADDR_ARG __addr,
            ^
utelnetd.c:597:23: warning: pointer targets in passing argument 1 of ‘remove_iacs’ differ in signedness [-Wpointer-sign]
     ptr = remove_iacs(ts->buf1 + ts->wridx1, maxlen, 
                       ^
utelnetd.c:190:1: note: expected ‘unsigned char *’ but argument is of type ‘char *’
 remove_iacs(unsigned char *bf, int len, int *processed, int *num_totty) {
 ^
gcc  -I. -pipe -DSHELLPATH=\"/bin/login\" -Wall -fomit-frame-pointer utelnetd.o  -o utelnetd
strip  --remove-section=.comment --remove-section=.note utelnetd
jeromesun@km:~/workshop/utelnetd-0.1.11$ ls
ChangeLog  LICENSE  Makefile  README  utelnetd  utelnetd.c  utelnetd.o
jeromesun@km:~/workshop/utelnetd-0.1.11$ ls -al
total 96
drwxr-xr-x  2 jeromesun jeromesun  4096 10月 26 14:10 .
drwxrwxr-x 17 jeromesun jeromesun  4096 10月 26 14:10 ..
-rw-r--r--  1 jeromesun jeromesun  2545 8月  11  2008 ChangeLog
-rw-r--r--  1 jeromesun jeromesun 17982 3月   6  2000 LICENSE
-rw-r--r--  1 jeromesun jeromesun  1120 8月  11  2008 Makefile
-rw-r--r--  1 jeromesun jeromesun   532 8月   8  2003 README
-rwxrwxr-x  1 jeromesun jeromesun 18912 10月 26 14:10 utelnetd
-rw-r--r--  1 jeromesun jeromesun 16430 8月   6  2003 utelnetd.c
-rw-rw-r--  1 jeromesun jeromesun 14600 10月 26 14:10 utelnetd.o
jeromesun@km:~/workshop/utelnetd-0.1.11$ 
```

## openbsd-inetd + telnetd 验证

虽然不用，但是存档。

### 问题分析

问题现象：telnet 客户端在几秒钟后自行退出。

```
[20181026-091148.403]a@xxx:~$ telnet localhost
[20181026-091148.436]Trying 127.0.0.1...
[20181026-091148.443]Connected to localhost.
[20181026-091148.444]Escape character is '^]'.
[20181026-091149.726]
[20181026-091153.446]Connection closed by foreign host.
```

log 分析:

```
Oct 26 18:17:42 xxx in.telnetd[4140]: connect from 127.0.0.1 (127.0.0.1)
Oct 26 18:17:42 xxx in.telnetd[4140]: error: cannot execute : No such file or directory
```

下载 telnetd 源码，grep 并没有发现文件有 `cannot execute` 的字样。
进一步下载 tcp-wrapper 和 openbsd-inetd 源码，发现该 log 在 `tcp-wrappers/tcpd.c` 中。应该是入参有问题，

```c
 /*
  * General front end for stream and datagram IP services. This program logs
  * the remote host name and then invokes the real daemon. For example,
  * install as /usr/etc/{tftpd,fingerd,telnetd,ftpd,rlogind,rshd,rexecd},
  * after saving the real daemons in the directory specified with the
  * REAL_DAEMON_DIR macro. This arrangement requires that the network daemons
  * are started by inetd or something similar. Connections and diagnostics
  * are logged through syslog(3).
  * 
  * Author: Wietse Venema, Eindhoven University of Technology, The Netherlands.
  */

#ifndef lint
static char sccsid[] = "@(#) tcpd.c 1.10 96/02/11 17:01:32";
#endif

/* System libraries. */

#include <sys/types.h>
#include <sys/param.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <stdio.h>
#include <syslog.h>
#include <string.h>

#ifndef MAXPATHNAMELEN
#define MAXPATHNAMELEN	BUFSIZ
#endif

#ifndef STDIN_FILENO
#define STDIN_FILENO	0
#endif

/* Local stuff. */

#include "patchlevel.h"
#include "tcpd.h"

int     allow_severity = SEVERITY;	/* run-time adjustable */
int     deny_severity = LOG_WARNING;	/* ditto */

main(argc, argv)
int     argc;
char  **argv;
{
    struct request_info request;
    char    path[MAXPATHNAMELEN];

    /* Attempt to prevent the creation of world-writable files. */

#ifdef DAEMON_UMASK
    umask(DAEMON_UMASK);
#endif

    /*
     * If argv[0] is an absolute path name, ignore REAL_DAEMON_DIR, and strip
     * argv[0] to its basename.
     */

    if (argv[0][0] == '/') {
	strcpy(path, argv[0]);
	argv[0] = strrchr(argv[0], '/') + 1;
    } else {
	sprintf(path, "%s/%s", REAL_DAEMON_DIR, argv[0]);
    }

    /*
     * Open a channel to the syslog daemon. Older versions of openlog()
     * require only two arguments.
     */

#ifdef LOG_MAIL
    (void) openlog(argv[0], LOG_PID, FACILITY);
#else
    (void) openlog(argv[0], LOG_PID);
#endif

    /*
     * Find out the endpoint addresses of this conversation. Host name
     * lookups and double checks will be done on demand.
     */

    request_init(&request, RQ_DAEMON, argv[0], RQ_FILE, STDIN_FILENO, 0);
    fromhost(&request);

    /*
     * Optionally look up and double check the remote host name. Sites
     * concerned with security may choose to refuse connections from hosts
     * that pretend to have someone elses host name.
     */

#ifdef PARANOID
    if (STR_EQ(eval_hostname(request.client), paranoid))
	refuse(&request);
#endif

    /*
     * The BSD rlogin and rsh daemons that came out after 4.3 BSD disallow
     * socket options at the IP level. They do so for a good reason.
     * Unfortunately, we cannot use this with SunOS 4.1.x because the
     * getsockopt() system call can panic the system.
     */

#ifdef KILL_IP_OPTIONS
    fix_options(&request);
#endif

    /*
     * Check whether this host can access the service in argv[0]. The
     * access-control code invokes optional shell commands as specified in
     * the access-control tables.
     */

#ifdef HOSTS_ACCESS
    if (!hosts_access(&request))
	refuse(&request);
#endif

    /* Report request and invoke the real daemon program. */

    syslog(allow_severity, "connect from %s", eval_client(&request));
    closelog();
    (void) execv(path, argv);
    syslog(LOG_ERR, "error: cannot execute %s: %m", path);  <---- 这时异常退出，关于 %m The ‘%m’ conversion prints the string corresponding to the error code in errno. https://stackoverflow.com/questions/20577557/whats-the-meaning-of-the-m-formatting-specifier .
    clean_exit(&request);
    /* NOTREACHED */
}
```

改下代码，把所有的 argv 打出来:

```
Oct 26 18:17:42 xxx in.telnetd[4140]: argc 0
Oct 26 18:17:42 xxx in.telnetd[4140]: 0: argv in.telnetd
Oct 26 18:17:42 xxx in.telnetd[4140]: connect from 127.0.0.1 (127.0.0.1)
Oct 26 18:17:42 xxx in.telnetd[4140]: error: cannot execute : No such file or directory
```

发现入参是 in.telnetd，不是 `/etc/inetd.conf` 中写的 `/usr/sbin/in.telnetd`。

```
telnet          stream  tcp     nowait  telnetd /usr/sbin/tcpd  /usr/sbin/in.telnetd
```

有点奇怪:

* openbsd-inetd 执行 telnet 服务时，执行的是什么命令
* 即使 openbsd-inetd 执行时，以 in.telnetd 直接传入，`REAL_DAEMON_DIR` 在编译时也有定义，应该有值才对。

```
cc -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -DFACILITY=LOG_DAEMON -DHOSTS_ACCESS  -DNETGROUP    -DDAEMON_UMASK=022 -DREAL_DAEMON_DIR=\"/usr/sbin\" -DPROCESS_OPTIONS -DACLEXEC -DKILL_IP_OPTIONS -DSEVERITY=LOG_INFO    -DRFC931_TIMEOUT=10  -DHOSTS_DENY=\"/etc/hosts.deny\" -DHOSTS_ALLOW=\"/etc/hosts.allow\"   -DSYS_ERRLIST_DEFINED -DHAVE_STRERROR -DHAVE_WEAKSYMS -DINET6=1 -Dss_family=__ss_family -Dss_len=__ss_len    -o tcpd.o -c tcpd.c
```

先不管了，直接改 tcpd.c 的代码，将所有 telnet 服务的入参强制变成 /usr/sbin/in.telnet:

```diff
--- <unnamed>
+++ <unnamed>
@@ -77,6 +77,11 @@
     (void) openlog(argv[0], LOG_PID);
 #endif
 
+    int i;
+    syslog(LOG_ERR, "argc %d", i);
+    for (i = 0; i < argc; i++) {
+        syslog(LOG_ERR, "%d: argv %s", i, argv[i]);
+    }
     /*
      * Find out the endpoint addresses of this conversation. Host name
      * lookups and double checks will be done on demand.
@@ -125,6 +130,13 @@
            eval_client(&request), eval_hostaddr(request.client));
 #else
     syslog(allow_severity, "connect from %s", eval_client(&request));
+#endif
+    if (strcmp(argv[0], "in.telnetd") == 0) {
+        syslog(LOG_ERR, "path before change: path %s, argv %s", path, argv[0]);
+        strcpy(path, "/usr/sbin/in.telnetd");
+        syslog(LOG_ERR, "path after change: path %s, argv %s", path, argv[0]);
+    }
+
     closelog();
     (void) execv(path, argv);
     syslog(LOG_ERR, "error: cannot execute %s: %m", path);
```

修改后正常:

```
[20181026-142113.815][jeromesun@64 ~]$ telnet 192.168.3.31
[20181026-142113.815]Trying 192.168.3.31...
[20181026-142113.816]Connected to 192.168.3.31.
[20181026-142113.817]Escape character is '^]'.
[20181026-142122.726]
[20181026-142133.853]Debian GNU/Linux 8 <---- 过了 20 秒才进来，看这个时间很像进行 dns 解析 timeout 了
[20181026-142133.892]
[20181026-142137.023]xxx login: a
[20181026-142137.352]Password: 
[20181026-142137.368]Last login: Fri Oct 26 19:46:52 CST 2018 from 11.1.1.99 on pts/10
[20181026-142137.698]Linux xxx 3.16.0-5-amd64 #1 SMP Debian 3.16.51-3+deb8u1 (2018-01-08) x86_64
[20181026-142137.700]Unauthorized access and/or use are prohibited.
[20181026-142137.700]All access and/or use are subject to monitoring.
[20181026-142137.700]
[20181026-142138.494]a@xxx:~$ 
[20181026-142200.714]a@xxx:~$ show version
[20181026-142201.869]Distribution: Debian 8.11
[20181026-142201.869]Kernel: 3.16.0-5-amd64
[20181026-142201.870]Build commit: 62a9f38
[20181026-142201.870]Build date: Thu Oct 25 20:56:07 CST 2018
...
[20181026-142202.045]
[20181026-142202.575]a@xxx:~$ 
[20181026-142203.202]telnet> q
[20181026-142203.203]Connection closed.
[20181026-142203.579][jeromesun@64 ~]$ 
```


### 编译记录

在源码根目录，执行 `dpkg-buildpackage -rfakeroot -b -us -uc -nc --as-root -Tbinary` 进行编译。

```
jeromesun@km:~/tcp-wrappers-7.6.q$ rm tcpd
jeromesun@km:~/tcp-wrappers-7.6.q$ rm tcpd.o 
jeromesun@km:~/tcp-wrappers-7.6.q$ rm debian/.stamp-build 
jeromesun@km:~/tcp-wrappers-7.6.q$ dpkg-buildpackage -rfakeroot -b -us -uc -nc --as-root -Tbinary
dpkg-buildpackage: source package tcp-wrappers
dpkg-buildpackage: source version 7.6.q-25
dpkg-buildpackage: source distribution unstable
dpkg-buildpackage: source changed by Marco d'Itri <md@linux.it>
dpkg-buildpackage: host architecture amd64
 fakeroot debian/rules binary
dh_testdir
/usr/bin/make  COPTS="-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2" LDOPTS="-Wl,-Bsymbolic-functions -Wl,-z,relro" linux
make[1]: Entering directory '/home/jeromesun/tcp-wrappers-7.6.q'
make[2]: Entering directory '/home/jeromesun/tcp-wrappers-7.6.q'
cc -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -DFACILITY=LOG_DAEMON -DHOSTS_ACCESS  -DNETGROUP    -DDAEMON_UMASK=022 -DREAL_DAEMON_DIR=\"/usr/sbin\" -DPROCESS_OPTIONS -DACLEXEC -DKILL_IP_OPTIONS -DSEVERITY=LOG_INFO    -DRFC931_TIMEOUT=10  -DHOSTS_DENY=\"/etc/hosts.deny\" -DHOSTS_ALLOW=\"/etc/hosts.allow\"   -DSYS_ERRLIST_DEFINED -DHAVE_STRERROR -DHAVE_WEAKSYMS -DINET6=1 -Dss_family=__ss_family -Dss_len=__ss_len    -o tcpd.o -c tcpd.c
tcpd.c:44:1: warning: return type defaults to ‘int’ [-Wimplicit-int]
 main(argc, argv)
 ^
tcpd.c: In function ‘main’:
tcpd.c:112:5: warning: implicit declaration of function ‘fix_options’ [-Wimplicit-function-declaration]
     fix_options(&request);
     ^
tcpd.c:141:12: warning: implicit declaration of function ‘execv’ [-Wimplicit-function-declaration]
     (void) execv(path, argv);
            ^
cc -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -DFACILITY=LOG_DAEMON -DHOSTS_ACCESS  -DNETGROUP    -DDAEMON_UMASK=022 -DREAL_DAEMON_DIR=\"/usr/sbin\" -DPROCESS_OPTIONS -DACLEXEC -DKILL_IP_OPTIONS -DSEVERITY=LOG_INFO    -DRFC931_TIMEOUT=10  -DHOSTS_DENY=\"/etc/hosts.deny\" -DHOSTS_ALLOW=\"/etc/hosts.allow\"   -DSYS_ERRLIST_DEFINED -DHAVE_STRERROR -DHAVE_WEAKSYMS -DINET6=1 -Dss_family=__ss_family -Dss_len=__ss_len    -Wl,-Bsymbolic-functions -Wl,-z,relro -o tcpd tcpd.o -Lshared -lwrap
make[2]: Leaving directory '/home/jeromesun/tcp-wrappers-7.6.q'
make[1]: Leaving directory '/home/jeromesun/tcp-wrappers-7.6.q'
touch debian/.stamp-build
test root = "`whoami`"
dh_testdir
dh_prep
dh_installdirs
dh_install
dh_installdocs --link-doc=libwrap0
dh_installdocs -p libwrap0 README
dh_installchangelogs CHANGES
dh_installman -p tcpd *.8
dh_installman -p libwrap0 *.5
dh_installman -p libwrap0-dev *.3
dh_link
mkdir -p /home/jeromesun/tcp-wrappers-7.6.q/debian/libwrap0/lib/x86_64-linux-gnu/ /home/jeromesun/tcp-wrappers-7.6.q/debian/libwrap0-dev/usr/lib/x86_64-linux-gnu/
cp shared/libwrap.so.0.7.6 /home/jeromesun/tcp-wrappers-7.6.q/debian/libwrap0/lib/x86_64-linux-gnu/
ln -s libwrap.so.0.7.6 /home/jeromesun/tcp-wrappers-7.6.q/debian/libwrap0/lib/x86_64-linux-gnu/libwrap.so.0
ln -s /lib/x86_64-linux-gnu/libwrap.so.0 \
        /home/jeromesun/tcp-wrappers-7.6.q/debian/libwrap0-dev/usr/lib/x86_64-linux-gnu/libwrap.so
mv /home/jeromesun/tcp-wrappers-7.6.q/debian/libwrap0-dev/usr/lib/libwrap.a /home/jeromesun/tcp-wrappers-7.6.q/debian/libwrap0-dev/usr/lib/x86_64-linux-gnu/
dh_link
dh_strip
dh_compress
dh_fixperms
dh_installdebconf
dh_makeshlibs -- -c4
dh_installdeb
dh_shlibdeps
dh_gencontrol
dh_md5sums
dh_builddeb
dpkg-deb: building package 'tcpd' in '../tcpd_7.6.q-25_amd64.deb'.
dpkg-deb: building package 'libwrap0' in '../libwrap0_7.6.q-25_amd64.deb'.
dpkg-deb: building package 'libwrap0-dev' in '../libwrap0-dev_7.6.q-25_amd64.deb'.
jeromesun@km:~/tcp-wrappers-7.6.q$ 
```

## xinetd + telnetd

未成功。
