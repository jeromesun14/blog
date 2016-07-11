title: GDB使用记录
date: 2015-06-17 20:40:18
toc: true
tags: [调试, GDB, Linux]
categories: programmer
keywords: [gdb, 调试, Debug, Linux]
description: 记录GDB使用过的方法。
---

简介
-------
GDB，GNU Debugger，[特性](http://en.wikipedia.org/wiki/Gdb#Features)如下：

* GDB具备各种调试功效，可对计算机程序的运行进行追踪、警告。使用者可以监控及修改程序内部变量的值，甚至可在程序的正常运行之外调用函数。
* GDB支持多数处理器架构
* 持续开发中
* 支持远程调试
* 支持内核调试，KGDB

<!--more-->

从事嵌入式软件开发两年来，主要在以下几方面使用GDB：

1. 查看、修改运行时变量；
2. 多线程调试，查看当前线程运行状态（以确定当前线程是不是因为等锁等原因挂起）；
3. 查看[coredump](http://zh.wikipedia.org/zh-cn/coredump)文件；
4. 碰到难缠的内存非法改写问题，用GDB的断点、物理watch功能查看内存变化以定位改写者；

> 引用公司一个技术牛人的话：*在大型的项目中，使用GDB的单步调试、软件watch是不现实的，因为会运行得实在太慢。*

命令小记：  

```
linux提示符
1. GDB进入正在运行的进程
    gdb 可执行文件 core文件 
    gdb -p pid
    
GDB提示符
1. 查看调用栈信息
    bt / backtrace / bt full
    frame n
    info locals
    info args

2. 查看、设置变量
    p 变量
    p 变量 = 新值
    set 变量 = 新值
    
3. 查看内存
    x/<n/f/u> <addr>

4. 线程调试
    info thread
    thread n
    thread apply all bt full
   
```

启动GDB
-------

### GCC选项###

想用GDB调试，则在GCC编译的时候要加上*-g*选项。

### 启动GDB ###

启动GDB的方法主要有以下几种：

* `gdb`
* `gdb executable_file`
* `gdb executable_file corefile`：查看coredump文件信息，定位coredump产生原因、触发源。
* `gdb attach pid`：调度运行时的进程或线程，同`gdb -p pid`。

### 善用help ###
在GDB提示符下输入`help`或`help 命令`，能够查看命令的帮助说明。

```
(gdb) help
List of classes of commands:

aliases -- Aliases of other commands
breakpoints -- Making program stop at certain points
data -- Examining data
files -- Specifying and examining files
internals -- Maintenance commands
obscure -- Obscure features
running -- Running the program
stack -- Examining the stack
status -- Status inquiries
support -- Support facilities
tracepoints -- Tracing of program execution without stopping the program
user-defined -- User-defined commands

Type "help" followed by a class name for a list of commands in that class.
Type "help all" for the list of all commands.
Type "help" followed by command name for full documentation.
Type "apropos word" to search for commands related to "word".
Command name abbreviations are allowed if unambiguous.
```

查看调用栈
-----------

写一个简单的例子（仅为样例，并不严谨）：

``` c
#include <stdbool.h>
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <assert.h>
#include <sys/prctl.h>

typedef struct {
    int member_a;
    int member_b;
} test_t;

int g_int;
bool g_bool;
char *g_str[] = {
    "Hello, GDB!",
    "It's funny."
};

void stay_here(int arg, test_t *test)
{
    int local;

    local = 100;
    while (true) {
    	local++;
	    if (local % 200 == 0) {
	        local = 0;
	    }
	    sleep(1);
    }

    return;
}

void *thread_process(void *arg)
{
    int in;
    char name[64];

    in = (int)arg;
    (void)snprintf(name, 64, "test-%d", in + 1);
    prctl(PR_SET_NAME, (unsigned long)name);  /* set thread name */

    while (true) {
	    sleep(2);
    }

    return NULL;
}

void create_thread(void)
{
    int i, rv;
    pthread_t tid;

    for (i = 0; i < 5; i++) {
    	rv = pthread_create(&tid, NULL, thread_process, (void *)i);
	    assert(rv == 0);
    }
}

int main(int argc, char **argv)
{
    int local;
    test_t test;

    local = 999;
    test.member_a = 10;
    test.member_b = 11;

    create_thread();
    stay_here(local, &test);

    return 0;
}

```

编译并运行起来，注意gcc的*-g*选项，这里使用*&*让程序运行到后台，*[1] 8043*指刚刚这个程序运行时的进程号，也可用`ps`命令查看。

```
sunnogo@a3e420:~/test/gdb$ gcc -o prt_mod_var prt_mod_var.c -g -Wall -lpthread
sunnogo@a3e420:~/test/gdb$ 
sunnogo@a3e420:~/test/gdb$ ls
prt_mod_var  prt_mod_var.c
sunnogo@a3e420:~/test/gdb$ ./prt_mod_var &
[1] 8043
sunnogo@a3e420:~/test/gdb$ 
sunnogo@a3e420:~/test/gdb$ ps -e | grep prt_mod_var
 8043 pts/1    00:00:00 prt_mod_var
```

接下来使用`gdb -p 8043`连入正在运行的进程中。还不明白为什么我的计算机中要求使用root权限才能让GDB attach到对应进程。

```
sunnogo@a3e420:~/test/gdb$ gdb -p 8043
GNU gdb (GDB) 7.5-ubuntu
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Attaching to process 8043
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
(gdb) quit
```

重新`sudo gdb -p pid`进入进程。

* 使用`bt`查看当前调用栈信息（**call stack**，即函数调用层次信息），当前进程的是由main() -> sleep() -> nanosleep() -> \_\_kernel\_vsyscall()一层一层调入。注意“*#数字*”，在GDB中这叫**stack frames**，或直接称为**frame**，运行栈由一个或多个连续的frame组成，数字越小代表调用层次越深。  
* 使用`bt full`查看详细调用栈信息，会把各个frame的入参和局部变量信息显示出来。这里bt是backtrace的缩写，GDB的全命令经常有其简短的写法。  


**注意**：GDB中，按回车默认是执行上一次命令。  
先MARK下面的“*No symbol table info available.*”

```
sunnogo@a3e420:~/test/gdb$ sudo gdb -p 8043
[sudo] password for sunnogo: 
GNU gdb (GDB) 7.5-ubuntu
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Attaching to process 8043
Reading symbols from /home/sunnogo/test/gdb/prt_mod_var...done.
Reading symbols from /lib/i386-linux-gnu/libc.so.6...(no debugging symbols found)...done.
Loaded symbols for /lib/i386-linux-gnu/libc.so.6
Reading symbols from /lib/ld-linux.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib/ld-linux.so.2
0xb7751424 in __kernel_vsyscall ()
(gdb) bt 
#0  0xb7751424 in __kernel_vsyscall ()
#1  0xb7640ce0 in nanosleep () from /lib/i386-linux-gnu/libc.so.6
#2  0xb7640aff in sleep () from /lib/i386-linux-gnu/libc.so.6
#3  0x0804845b in stay_here (arg=999, test=0xbf8e5118) at prt_mod_var.c:26
#4  0x08048492 in main (argc=1, argv=0xbf8e51c4) at prt_mod_var.c:41
(gdb) bt full
#0  0xb7751424 in __kernel_vsyscall ()
No symbol table info available.
#1  0xb7640ce0 in nanosleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#2  0xb7640aff in sleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#3  0x0804845b in stay_here (arg=999, test=0xbf8e5118) at prt_mod_var.c:26
        local = 113
#4  0x08048492 in main (argc=1, argv=0xbf8e51c4) at prt_mod_var.c:41
        local = 999
        test = {member_a = 10, member_b = 11}


```

* 使用`frame n`进入“*#n*”的frame。默认显示当前函数名、函数入参、当前运行处所在源文件的代码行位置，并显示当前行代码。
* 使用`info`命令查看frame详细信息，info命令不是全命令，后面还有子命令。info有很多子命令，除本frame外，还可以查看本进程信息、系统信息，这里仅仅是冰山一角。
  + `info frame` 显示当前frame信息
  + `info args` 显示入参信息
  + `info local` 显示局部变量信息

```
(gdb) frame 3
#3  0x0804845b in stay_here (arg=999, test=0xbf8e5118) at prt_mod_var.c:26
26              sleep(1);
(gdb) info frame
Stack level 3, frame at 0xbf8e5100:
 eip = 0x804845b in stay_here (prt_mod_var.c:26); saved eip 0x8048492
 called by frame at 0xbf8e5130, caller of frame at 0xbf8e50d0
 source language c.
 Arglist at 0xbf8e50f8, args: arg=999, test=0xbf8e5118
 Locals at 0xbf8e50f8, Previous frame's sp is 0xbf8e5100
 Saved registers:
  ebx at 0xbf8e50f4, ebp at 0xbf8e50f8, eip at 0xbf8e50fc
(gdb) info args
arg = 999
test = 0xbf8e5118
(gdb) info local
local = 113
(gdb) 
```

查看、修改变量
---------------
`p var`查看变量信息，p是print的缩写。
  + `p var`
  + `p *(指针类型)地址`
  + `p *结构体指针`
  + `p 数组名`

```
# 打印变量 
(gdb) p g_int
$3 = 0
(gdb) p g_bool
$4 = false

# 打印特定类型指针
(gdb) info local
local = 113
(gdb) p &local
$11 = (int *) 0xbf8e50ec
(gdb) p *(int *) 0xbf8e50ec
$12 = 113
(gdb) 

# 打印结构体指针
(gdb) p test
$1 = (test_t *) 0xbf8e5118
(gdb) p *test
$2 = {member_a = 10, member_b = 11}

# 打印数组名
(gdb) p g_str
$5 = {0x8048538 "Hello, GDB!", 0x8048544 "It's funny."}
(gdb) p g_str[0]
$6 = 0x8048538 "Hello, GDB!"

```

`print`不仅可以用来查看变量，还可用于设置变量。`print var=value`。  
设置变量值的命令还有`set`，`set var=value`。

```
# set int
(gdb) print local
$1 = 109
(gdb) print local=20
$2 = 20
(gdb) print local
$3 = 20
(gdb) set local=30
(gdb) print local
$4 = 30

# set bool
(gdb) print g_bool
$5 = false
(gdb) set g_bool=true
No symbol "true" in current context.
(gdb) set g_bool=1
(gdb) print g_bool
$6 = true

# set pointer
(gdb) print g_str
$7 = {0x8048538 "Hello, GDB!", 0x8048544 "It's funny."}
(gdb) set g_str[0]="SETTING VAR"
(gdb) print g_str
$8 = {0x8e05008 "SETTING VAR", 0x8048544 "It's funny."}
(gdb) 

```

查看内存
--------

`examine`查看内存，缩写是`x`。命令格式：

```
x/<n/f/u> <addr>
```
n、f、u是可选参数，说明如下：

```
(gdb) help x
Examine memory: x/FMT ADDRESS.
ADDRESS is an expression for the memory address to examine.
FMT is a repeat count followed by a format letter and a size letter.
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
  t(binary), f(float), a(address), i(instruction), c(char) and s(string).
Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).
The specified number of objects of the specified size are printed
according to the format.

Defaults for format and size letters are those previously used.
Default count is 1.  Default address is following last thing printed
with this command or "print".
```

* `n`表示要打印的多少个单位的内存，默认是1，单位由`u`定义；
* `f`表示打印的格式，格式有：
  + *o*，octal，八进制；
  + *x*，hex，十六进制；
  + *d*，decimal，十进制；
  + *u*，unsigned decimal，无符号十进制；
  + *t*，binary，二进制；
  + *f*，float；
  + *a*，address；
  + *i*，instruction，指令；
  + *c*，char，字符；
  + *s*，string，字符串。
* `u`定义单位，*b*表示1字节，*h*表示2字节，*w*表示4字节，*g*表示8字节。

```
# 当前CPU是intel i3，小端

# 以十进制形式打印
(gdb) x/8db test
0xbf8e5118:     10      0       0       0       11      0       0       0
(gdb) x/4dh test
0xbf8e5118:     10      0       11      0
(gdb) x/2dw test
0xbf8e5118:     10      11
(gdb) x/2d test
0xbf8e5118:     10      11
(gdb) x/1dg test
0xbf8e5118:     47244640266  # 注意和x/1xg test的结果比较

# 以二进制形式打印
(gdb) x/1tg test
0xbf8e5118:     0000000000000000000000000000101100000000000000000000000000001010
(gdb) x/2tw test
0xbf8e5118:     00000000000000000000000000001010        00000000000000000000000000001011
(gdb) x/4th test
0xbf8e5118:     0000000000001010        0000000000000000        0000000000001011        0000000000000000
(gdb) x/8tb test
0xbf8e5118:     00001010        00000000        00000000        00000000        00001011        0000000000000000        00000000

# 以十六进制形式打印
(gdb) x/8xb test
0xbf8e5118:     0x0a    0x00    0x00    0x00    0x0b    0x00    0x00    0x00
(gdb) x/4xh test
0xbf8e5118:     0x000a  0x0000  0x000b  0x0000
(gdb) x/2xw test
0xbf8e5118:     0x0000000a      0x0000000b
(gdb) x/1xg test
0xbf8e5118:     0x0000000b0000000a

# 打印字符或字符串
(gdb) x/30cb g_str[0]
0x8048538:      72 'H'  101 'e' 108 'l' 108 'l' 111 'o' 44 ','  32 ' '  71 'G'
0x8048540:      68 'D'  66 'B'  33 '!'  0 '\000'        73 'I'  116 't' 39 '\'' 115 's'
0x8048548:      32 ' '  102 'f' 117 'u' 110 'n' 110 'n' 121 'y' 46 '.'  0 '\000'
0x8048550:      1 '\001'        27 '\033'       3 '\003'        59 ';'  56 '8'  0 '\000'
(gdb) x/s g_str[0]
0x8048538:      "Hello, GDB!"
```

查看线程信息
------------
有两种方法可以进入线程调试：

* 设置线程名，用ps查看母进程的线程信息，获取tid，再启动GDB进入；
* 直接启动GDB调试母进程，`info thread`查看所有线程信息，获取到想要的线程的GDB内部编号n，`thread n`进入线程的调用栈。

### 直接获取、调试线程 ###

上面样例中创建5条线程，并使用*prctl*函数为每条线程命名为"test-n"。
这样可以通过`ps -eL | grep test（或者test进程的pid）`来查看刚创建的线程的tid。然后`gdb -p tid`进入线程调度。这里进入编号为4的线程。

```
sunnogo@a3e420:~/test/gdb$ gcc -o test test.c -g -Wall -lpthread
sunnogo@a3e420:~/test/gdb$ ./test &
[2] 16427
sunnogo@a3e420:~/test/gdb$ ps -eL | grep test
16427 16427 pts/1    00:00:00 test
16427 16428 pts/1    00:00:00 test-1
16427 16429 pts/1    00:00:00 test-2
16427 16430 pts/1    00:00:00 test-3
16427 16431 pts/1    00:00:00 test-4
16427 16432 pts/1    00:00:00 test-5
sunnogo@a3e420:~/test/gdb$ sudo gdb -p 16431
GNU gdb (GDB) 7.5-ubuntu
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Attaching to process 16431

warning: process 16431 is a cloned process
Reading symbols from /home/sunnogo/test/gdb/test...done.
Reading symbols from /lib/i386-linux-gnu/libpthread.so.0...(no debugging symbols found)...done.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/i386-linux-gnu/libthread_db.so.1".
Loaded symbols for /lib/i386-linux-gnu/libpthread.so.0
Reading symbols from /lib/i386-linux-gnu/libc.so.6...(no debugging symbols found)...done.
Loaded symbols for /lib/i386-linux-gnu/libc.so.6
Reading symbols from /lib/ld-linux.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib/ld-linux.so.2
0xb774f424 in __kernel_vsyscall ()
(gdb) bt full
#0  0xb774f424 in __kernel_vsyscall ()
No symbol table info available.
#1  0xb7623d06 in nanosleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#2  0xb7623aff in sleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#3  0x080485ee in thread_process (arg=0x3) at test.c:46
        in = 3
        name = "test-4", '\000' <repeats 57 times>
#4  0xb771cd4c in start_thread () from /lib/i386-linux-gnu/libpthread.so.0
No symbol table info available.
#5  0xb765abae in clone () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
(gdb) 
```

### 间接获取、调试线程 ###

注意和上一种方法的对比，相比起来，第一种方法要方便得多。也从侧面看出为每个线程命名的重要性。

```
sunnogo@a3e420:~/test/gdb$ 
nnogo@a3e420:~/test/gdb$ sudo gdb attach 16427
GNU gdb (GDB) 7.5-ubuntu
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
attach: No such file or directory.
Attaching to process 16427
Reading symbols from /home/sunnogo/test/gdb/test...done.
Reading symbols from /lib/i386-linux-gnu/libpthread.so.0...(no debugging symbols found)...done.
[New LWP 16432]
[New LWP 16431]
[New LWP 16430]
[New LWP 16429]
[New LWP 16428]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/i386-linux-gnu/libthread_db.so.1".
Loaded symbols for /lib/i386-linux-gnu/libpthread.so.0
Reading symbols from /lib/i386-linux-gnu/libc.so.6...(no debugging symbols found)...done.
Loaded symbols for /lib/i386-linux-gnu/libc.so.6
Reading symbols from /lib/ld-linux.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib/ld-linux.so.2
0xb774f424 in __kernel_vsyscall ()
(gdb) info thread
  Id   Target Id         Frame 
  6    Thread 0xb7568b40 (LWP 16428) "test-1" 0xb774f424 in __kernel_vsyscall ()
  5    Thread 0xb6d67b40 (LWP 16429) "test-2" 0xb774f424 in __kernel_vsyscall ()
  4    Thread 0xb6566b40 (LWP 16430) "test-3" 0xb774f424 in __kernel_vsyscall ()
  3    Thread 0xb5d65b40 (LWP 16431) "test-4" 0xb774f424 in __kernel_vsyscall ()
  2    Thread 0xb5564b40 (LWP 16432) "test-5" 0xb774f424 in __kernel_vsyscall ()
* 1    Thread 0xb75696c0 (LWP 16427) "test" 0xb774f424 in __kernel_vsyscall ()
(gdb) thread 3
[Switching to thread 3 (Thread 0xb5d65b40 (LWP 16431))]
#0  0xb774f424 in __kernel_vsyscall ()
(gdb) bt full
#0  0xb774f424 in __kernel_vsyscall ()
No symbol table info available.
#1  0xb7623d06 in nanosleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#2  0xb7623aff in sleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#3  0x080485ee in thread_process (arg=0x3) at test.c:46
        in = 3
        name = "test-4", '\000' <repeats 57 times>
#4  0xb771cd4c in start_thread () from /lib/i386-linux-gnu/libpthread.so.0
No symbol table info available.
#5  0xb765abae in clone () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
(gdb) q
A debugging session is active.

        Inferior 1 [process 16427] will be detached.

Quit anyway? (y or n) y
Detaching from program: /home/sunnogo/test/gdb/test, process 16427
sunnogo@a3e420:~/test/gdb$ 
sunnogo@a3e420:~/test/gdb$ 

```

### 查看所有线程堆栈 ###
使用 `thread apply all bt full`，查看所有线程的堆栈，如果线程多，可能会产生短暂刷屏。

gdb中调用调用函数
-------
`call func_name(param1, param2, ...)`，目前还没有明白如果参数是结构体要怎么整。注意，只能在进程上下文中才能使用，coredump中无法使用。

gdb中申请内存
-------
`p malloc(size)`，结果会返回一个指针，即可正常使用这个指针。注意，只能在进程上下文中才能使用，coredump中无法使用。如下例：

```
(gdb) p malloc(4)
[New Thread 0x693ff460 (LWP 2033)]
[Switching to Thread 0xb6101000 (LWP 1456)]
$1 = (void *) 0xb58d01e0    <----使用这个返回的指针。
```

查看寄存器信息
-----------------
to-do

GDB反汇编
-----------
to-do

断点设置
------
to-do

内存监控
--------
to-do

GCC选项对GDB的影响
------------------

### GCC -g选项的影响 ###

注意上面的，如果gcc编译的时候不加`-g`选项，那么frame 3也会显示“*No symbol table info available.*”，无符号表信息可用，全局变量g_str也打不出来。

```
(gdb) bt full
#0  0xb77a3424 in __kernel_vsyscall ()
No symbol table info available.
#1  0xb7692ce0 in nanosleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#2  0xb7692aff in sleep () from /lib/i386-linux-gnu/libc.so.6
No symbol table info available.
#3  0x08048462 in main ()
No symbol table info available.
(gdb) p g_str
$1 = 134513928
(gdb) p g_str[0]
cannot subscript something of type `<data variable, no debug info>'
(gdb) p g_bool
$2 = 0
(gdb) 
```

### GCC -fomit-frame-pointer选项的影响 ###

