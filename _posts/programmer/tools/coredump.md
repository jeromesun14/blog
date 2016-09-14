Coredump文件简易指南
====================

coredump简介
------------
通常情况下，coredump（亦称为core文件）文件包含程序运行时的内存信息，含寄存器状态、堆栈指针、内存管理信息、操作系统flags及其他信息，可以理解为把程序工作的当前状态存储成一个文件。Coredump文件通常于程序异常终止（crashed）时自动生成，常用于辅助分析和解决bug。可简单地理解为，10.x RGOS跑飞的时候打印的一系列信息，都保存在这个coredump文件里面，可通过这个coredump文件进行栈回溯和反汇编。

Coredump文件在不同的操作系统上文件类型不一，在目前的Unix-like操作系统中，使用ELF类型保存coredump文件，如下所示。目前常见阅读coredump文件的工具有BFD（GNU Binutils Binary File Descriptor library），以及使用这个库的GDB（GNU Debugger）和objdump。下文使用GDB阅读coredump文件。
```
rj@rj:~/test/coredump$ file core-cd_test-1359224646
core-cd_test-1359224646: ELF 32-bit LSB core file Intel 80386, version 1 (SYSV), SVR4-style, from './cd_test'
```

产生原因
--------
产生coredump文件的原因有很多，主要为低级bug中的内存访问越界、使用空指针、堆栈溢出等。用coredump文件来查找此类bug效率极高。

coredump相关系统设置
-------------------
以Ubuntu 12.04为例。
### 系统是否允许产生coredump文件
Coredump也是类似消息队列，属于系统资源。使用“ulimit –a”查看系统资源限制，如图1所示。对于coredump文件，使用“ulimit -c”查看其限制，如果为0，表示系统不会产生coredump文件。

Ubuntu系统可查看/etc/security/limits.conf查看系统资源限制相关的单位。如下所示，可在这进行长久的设置，因其他系统资源项也是平时我们需要注意的，因此在此把整个limits.conf文件copy过来。可用“ulimit -c your-size”进行临时设置，不过重启后无效。
```
sunnogo@R04220:~$ uname -a
Linux R04220 4.4.0-36-generic #55-Ubuntu SMP Thu Aug 11 18:01:55 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
sunnogo@R04220:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15232
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15232
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

配置文件limits.conf：
```
sunnogo@R04220:~$ cat /etc/security/limits.conf
# /etc/security/limits.conf
#
#Each line describes a limit for a user in the form:
#
#<domain>        <type>  <item>  <value>
#
#Where:
#<domain> can be:
#        - a user name
#        - a group name, with @group syntax
#        - the wildcard *, for default entry
#        - the wildcard %, can be also used with %group syntax,
#                 for maxlogin limit
#        - NOTE: group and wildcard limits are not applied to root.
#          To apply a limit to the root user, <domain> must be
#          the literal username root.
#
#<type> can have the two values:
#        - "soft" for enforcing the soft limits
#        - "hard" for enforcing hard limits
#
#<item> can be one of the following:
#        - core - limits the core file size (KB)
#        - data - max data size (KB)
#        - fsize - maximum filesize (KB)
#        - memlock - max locked-in-memory address space (KB)
#        - nofile - max number of open files
#        - rss - max resident set size (KB)
#        - stack - max stack size (KB)
#        - cpu - max CPU time (MIN)
#        - nproc - max number of processes
#        - as - address space limit (KB)
#        - maxlogins - max number of logins for this user
#        - maxsyslogins - max number of logins on the system
#        - priority - the priority to run user process with
#        - locks - max number of file locks the user can hold
#        - sigpending - max number of pending signals
#        - msgqueue - max memory used by POSIX message queues (bytes)
#        - nice - max nice priority allowed to raise to values: [-20, 19]
#        - rtprio - max realtime priority
#        - chroot - change root to directory (Debian-specific)
#
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#root            hard    core            100000
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#ftp             -       chroot          /ftp
#@student        -       maxlogins       4

# End of file
sunnogo@R04220:~$
```

### coredump文件的保存路径和文件名格式
本节内容完全来自[这里](http://www.ms2006.com/archives/151)。

查看正在使用的core文件路径和格式【more /proc/sys/kernel/core_pattern】。自动添加pid的配置是在【more /proc/sys/kernel/core_uses_pid】里面配置的，如果为1就是自动添加。

修改/etc/sysctl.conf文件【vi /etc/sysctl.conf】，添加需要保存的路径【kernel.core_pattern = /tmp/corefile/core.%e.%t】，需要注意的是该路径必须应用有写的权限，不然core文件是不会生成的。再执行命令【sysctl -p】即可生效。关于core_users_pid默认在sysctl文件里面已经存在，不需要更改，pid还是很重要的信息。

附上core文件支持的格式列表：
```
%p – insert pid into filename 【pid】
%u – insert current uid into filename 【uid】
%g – insert current gid into filename 【gid】
%s – insert signal that caused the coredump into the filename 【core信号】
%t – insert UNIX time that the coredump occurred into filename 【core文件生成时的unix时间】
%h – insert hostname where the coredump happened into filename 【主机名】
%e – insert coredumping executable name into filename 【应用的名字】
```

可自己写个除0或者访问NULL指针的程序来验证coredump设置OK。

如何“阅读”coredump文件
----------------------
这章是这个指南的核心所在。编译时的Gcc选项会影响coredump文件，目前知道的有“-g”选项（用于调试，能够在gdb中看到所有的symbol，即函数、变量等）和“-fomit-frame-pointer”（用于优化栈顶指针，将不会保留ebp的信息）。

【Tips】gdb命令也可用tab键补全。

以如下程序为例：
```
cd_test.c：

static int g_t1;
static int g_t2 = 2;

int div(int div_i, int div_j)
{
    int a4, b4;
    char *c4;
    
    a4 = div_i + 3;
    b4 = div_j + 3;
    c4 = "divf";

    return (div_i / div_j);
}

int sub(int sub_i, int sub_j)
{
    int a3, b3;
    char *c3;

    a3 = sub_i + 2;
    b3 = sub_j + 2;
    c3 = "subf";
    div(a3, 0);    // 这里通过除0让程序运行时挂掉

    return (sub_i - sub_j);
}

int add(int add_i, int add_j)
{
    int a2, b2;
    char *c2;

    a2 = add_i + 1;
    b2 = add_j + 1;
    c2 = "addf";
    sub(a2, b2);

    return (add_i + add_j);
}

int main(int argc, char *argv[])
{
    int a1, b1;
    char *c1;
    static int u1;
    static int x1 = 1;

    a1 = 1;
    b1 = 0;
    c1 = "main function";

    add(a1, b1);

    return 0;
}
```

编译，执行。下文会涉及到三种gcc编译选项：默认的“gcc –g”，不带“-g”选项，“gcc -g -fomit-frame-pointer”。
```
rj@rj:~/test/coredump$ gcc -g -o cd_test cd_test.c
rj@rj:~/test/coredump$ ./cd_test
浮点数例外 (核心已转储)    提示产生coredump文件
```

* 运行gdb阅读core文件，命令为“gdb 程序 对应coredump文件”，这时就进入gdb的提示符“(gdb)”。
```
rj@rj:~/test/coredump$ gdb cd_test core-cd_test-1359266323
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /home/rj/test/coredump/cd_test...done.
[New LWP 28375]

warning: Can't read pathname for load map: 输入/输出错误.
Core was generated by `./cd_test'.
Program terminated with signal 8, Arithmetic exception.
#0  0x080483eb in div (div_i=4, div_j=0) at cd_test.c:13
13          return (div_i / div_j);
(gdb)
```
* 执行“backtrace”或其缩写“bt”进入栈回溯。如下所示，函数调用的顺序是从#n中数值高的往低的。比如这些是main()add()sub()div()，#0表示跑飞的地方。英文用frame表示一个调用层次，下文所说的各种信息对应某个指定frame。
```
(gdb) bt
#0  0x080483eb in div (div_i=4, div_j=0) at cd_test.c:13
#1  0x08048422 in sub (sub_i=2, sub_j=1) at cd_test.c:24
#2  0x08048461 in add (add_i=1, add_j=0) at cd_test.c:37
#3  0x08048498 in main (argc=1, argv=0xbfba3264) at cd_test.c:53
```
若编译时没有-g选项，则结果是：
```
Core was generated by `./cd_test'.
Program terminated with signal 8, Arithmetic exception.
#0  0x080483eb in div ()
(gdb) bt
#0  0x080483eb in div ()    对比带-g选项的少了些什么？
#1  0x08048422 in sub ()
#2  0x08048461 in add ()
#3  0x08048498 in main ()
```
* 执行“frame n”进入对应调用层次。以frame 3为例。
```
(gdb) frame 3
#3  0x08048498 in main (argc=1, argv=0xbfba3264) at cd_test.c:53
53          add(a1, b1);
```
没带-g选项的：
```
(gdb) frame 3
#3  0x08048498 in main ()    少了很多信息
```
* 执行“info frame”查看本层调用的栈信息
```
(gdb) info frame
Stack level 3, frame at 0xbfba31d0:
 eip = 0x8048498 in main (cd_test.c:53); saved eip 0xb75d24d3
 caller of frame at 0xbfba31b0
 source language c.
 Arglist at 0xbfba31c8, args: argc=1, argv=0xbfba3264
 Locals at 0xbfba31c8, Previous frame's sp is 0xbfba31d0
 Saved registers:
  ebp at 0xbfba31c8, eip at 0xbfba31cc
```

带-fomit-frame-pointer选项：
```
(gdb) info frame
Stack level 3, frame at 0xbffa57c0:
 eip = 0x80484af in main (cd_test.c:53); saved eip 0xb753d4d3
 caller of frame at 0xbffa57a4
 source language c.
 Arglist at unknown address.
 Locals at unknown address, Previous frame's sp is 0xbffa57c0
 Saved registers:
  eip at 0xbffa57bc    没有ebp
```

不带-g选项：
```
(gdb) info frame
Stack level 3, frame at 0xbfca4a90:
 eip = 0x8048498 in main; saved eip 0xb75e74d3
 caller of frame at 0xbfca4a70
 Arglist at 0xbfca4a88, args:
 Locals at 0xbfca4a88, Previous frame's sp is 0xbfca4a90
 Saved registers:
  ebp at 0xbfca4a88, eip at 0xbfca4a8c
```

* 使用“info args”查看参数信息
```
(gdb) info args
argc = 1
argv = 0xbfba3264
```
带-fomit-frame-pointer同上。
不带-g选项：
```
(gdb) info args
No symbol table info available.
```

* 使用“info locals”查看所有局部变量信息
```
(gdb) info locals
a1 = 1
b1 = 0
c1 = 0x804857f "main function"
u1 = 0
x1 = 1
```
带-fomit-frame-pointer同上。
不带-g选项依旧是“No symbol table info available”。

* 使用“print 变量名”查看变量信息，比如查看全局变量。实验结果显示，不管带不带-g或-fomit-pointer-frame选项，全局变量都能查看得到。
```
(gdb) print g_t1
$1 = 0
```

* 使用“info registers”查看寄存器信息，三个选项结果一样。`info all-registers`会打印更多寄存器信息。
```
(gdb) info registers
eax            0x4      4
ecx            0xbfba3264       -1078316444
edx            0x0      0
ebx            0xb775eff4       -1217007628
esp            0xbfba31b0       0xbfba31b0
ebp            0xbfba31c8       0xbfba31c8
esi            0x0      0
edi            0x0      0
eip            0x8048498        0x8048498 <main+45>
eflags         0x10246  [ PF ZF IF RF ]
cs             0x73     115
ss             0x7b     123
ds             0x7b     123
es             0x7b     123
fs             0x0      0
gs             0x33     51
```

* 使用“info threads”查看线程信息，多线程的暂未实验。
```
(gdb) info threads
  Id   Target Id         Frame
* 1    LWP 28375         0x08048498 in main (argc=1, argv=0xbfba3264) at cd_test.c:53
```
无-g选项的信息会少一些。
```
(gdb) info threads
  Id   Target Id         Frame
* 1    LWP 28229         0x08048498 in main ()
```

* 使用“info macro”查看宏，暂未实验。
* 使用“disassemble”反汇编。还可以通过“disassemble 函数名”对指定函数进行反汇编。
```
(gdb) disassemble
Dump of assembler code for function main:
   0x0804846b <+0>:     push   %ebp
   0x0804846c <+1>:     mov    %esp,%ebp
   0x0804846e <+3>:     sub    $0x18,%esp
   0x08048471 <+6>:     movl   $0x1,-0xc(%ebp)
   0x08048478 <+13>:    movl   $0x0,-0x8(%ebp)
   0x0804847f <+20>:    movl   $0x804857f,-0x4(%ebp)
   0x08048486 <+27>:    mov    -0x8(%ebp),%eax
   0x08048489 <+30>:    mov    %eax,0x4(%esp)
   0x0804848d <+34>:    mov    -0xc(%ebp),%eax
   0x08048490 <+37>:    mov    %eax,(%esp)
   0x08048493 <+40>:    call   0x8048430 <add>
=> 0x08048498 <+45>:    mov    $0x0,%eax    标识在哪挂掉
   0x0804849d <+50>:    leave
   0x0804849e <+51>:    ret
End of assembler dump.
```
不带-g选项的同上。
带-fomit-frame-pointer：
```
(gdb) disassemble
Dump of assembler code for function main:    没有ebp了
   0x08048480 <+0>:     sub    $0x18,%esp
   0x08048483 <+3>:     movl   $0x1,0xc(%esp)
   0x0804848b <+11>:    movl   $0x0,0x10(%esp)
   0x08048493 <+19>:    movl   $0x804859f,0x14(%esp)
   0x0804849b <+27>:    mov    0x10(%esp),%eax
   0x0804849f <+31>:    mov    %eax,0x4(%esp)
   0x080484a3 <+35>:    mov    0xc(%esp),%eax
   0x080484a7 <+39>:    mov    %eax,(%esp)
   0x080484aa <+42>:    call   0x804843d <add>
=> 0x080484af <+47>:    mov    $0x0,%eax
   0x080484b4 <+52>:    add    $0x18,%esp
   0x080484b7 <+55>:    ret
End of assembler dump.
```

* Gdb中使用Linux命令：`shell 你的命令`。
```
(gdb) shell ls -al /usr
总用量 164
drwxr-xr-x  10 root root  4096  4月 23  2012 .
drwxr-xr-x  23 root root  4096  1月 27 01:25 ..
drwxr-xr-x   2 root root 61440  1月 19 02:45 bin
drwxr-xr-x   2 root root  4096  9月  5 19:53 games
drwxr-xr-x  37 root root 20480 11月 28 14:20 include
drwxr-xr-x 194 root root 36864  1月  8 19:31 lib
drwxr-xr-x  11 root root  4096  6月  1  2012 local
drwxr-xr-x   2 root root 12288  1月 22 12:29 sbin
drwxr-xr-x 316 root root 12288  1月 22 12:29 share
drwxr-xr-x  10 root root  4096 12月 11 20:32 src
```

其他
----
GDB的功能远不止于此，本文档只是个简易的coredump文件阅读指南。

另外，关于-fomit-frame-pointer选项，在[gdb官方说明文档](http://sourceware.org/gdb/current/onlinedocs/gdb/Frames.html#Frames)： 
> Some compilers provide a way to compile functions so that they operate without stack frames. (For example, the gcc option `-fomit-frame-pointer' generates functions without a frame.) **This is occasionally done with heavily used library functions to save the frame setup time.** gdb has limited facilities for dealing with these function invocations. If the innermost function invocation has no stack frame, gdb nevertheless regards it as though it had a separate frame, which is numbered zero as usual, allowing correct tracing of the function call chain. However, gdb has no provision for frameless functions elsewhere in the stack.

上面的黑体字说明-fomit-frame-pointer会造成编译器有时会把函数编译成无stack frame，即有可能在backtrace的出现各种“?? ()”，

如下所示。
```
(gdb) bt
#0  0x080483eb in div (div_i=134513698, div_j=4) at cd_test.c:13
#1  0xbfec16a8 in ?? ()
#2  0x08048422 in sub (sub_i=134513761, sub_j=2) at cd_test.c:24
#3  0xbfec16c8 in ?? ()
#4  0x08048461 in add (add_i=134513816, add_j=1) at cd_test.c:37
#5  0xbfec16e8 in ?? ()
#6  0x08048498 in main (argc=-1218612013, argv=0x1) at cd_test.c:51
```

