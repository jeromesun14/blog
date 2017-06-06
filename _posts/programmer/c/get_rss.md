
title: linux c 查看当前进程内存使用值
date: 2017-06-06 09:28:24
toc: true
tags: [linux, c]
categories: c
keywords: [linux, c, memory, 内存, 进程, process, proc, meminfo, rss, status]
description: Linux c 如何查看当前进程内存使用情况。
---

## 问题背景
排查大进程的内存使用率过高问题。
在初始化阶段排查每个业务初始化前后内存占用情况，对在初始化阶段动态分配内存的情况有效，对运行时动态分配内存无效。

## 思路

* 获取进程 pid
* 查看 `/proc/[pid]/status`，获取 VmRSS 值
* 截取 VmRSS 行，打印 VmRSS 值
* 通过 sed/awk 获取打印值，并进行计算，确认哪个业务初始化消耗太多内存

### /proc/[pid]/status

详见 [man 5 proc](http://man7.org/linux/man-pages/man5/proc.5.html)。

> VmRSS: Resident set size.  Note that the value here is the sum of RssAnon, RssFile, and RssShmem.

样例：

```
 sunyongfeng  ~  cat /proc/1610/status 
Name:	upstart-udev-br
State:	S (sleeping)
Tgid:	1610
Ngid:	0
Pid:	1610
PPid:	1524
TracerPid:	0
Uid:	1003	1003	1003	1003
Gid:	1003	1003	1003	1003
FDSize:	64
Groups:	27 999 1003 
VmPeak:	   34596 kB
VmSize:	   34596 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	    1916 kB
VmRSS:	    1788 kB
VmData:	     240 kB
VmStk:	     136 kB
VmExe:	      80 kB
VmLib:	    4760 kB
VmPTE:	      76 kB
VmSwap:	     204 kB
Threads:	1
SigQ:	1/15283
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000000001
SigCgt:	0000000180014000
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	0000003fffffffff
Seccomp:	0
Cpus_allowed:	ff
Cpus_allowed_list:	0-7
Mems_allowed:	00000000,00000001
Mems_allowed_list:	0
voluntary_ctxt_switches:	2212
nonvoluntary_ctxt_switches:	30
```

## 代码

```c
const char data_mem[] = "VmRSS:";
 
/* 打印当前消耗内存 */
void print_mem(int pid, char *func_name)
{
    FILE *stream;
    char cache[256];
    char mem_info[64];

    printf("after func:[%-30s]\t", func_name);
    sprintf(mem_info, "/proc/%d/status", pid);
    stream = fopen(mem_info, "r");
    if (stream == NULL) {
        return;
    }

    while(fscanf(stream, "%s", cache) != EOF) {
        if (strncmp(cache, data_mem, sizeof(data_mem)) == 0) {
            if (fscanf(stream, "%s", cache) != EOF) {
                printf("hw memory[%s]<=======\n", cache);
                break;
            }
        } 
    }
}
```

调用：

```
print_mem(getpid(), your_func_name);
```
