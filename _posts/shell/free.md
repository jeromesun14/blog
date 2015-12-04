title: Linux命令行查看当前内存使用情况 - free 
date: 2015-04-13 22:31:59
tags: [Linux, 内存管理]
categories: shell
toc: true
keywords: [linux, command, free, memory management]
description: Linux命令行查看当前内存使用情况。
---

free - Display amount of free and used memory in the system.

## 常见选项和样例

* `-h`，即`--human-readable`，显示的单位为大家熟悉的K/M/G等。
* `-s second`，持续显示内存使用情况，每隔second秒输出一次。
* `-c count`，与`-s`同时使用，表示持续输出的次数。

<!--more-->

```
1) 不带参数时，输出的单位是byte。
sunnogo@a3e420:~$ free
             total       used       free     shared    buffers     cached
Mem:       4039288    2005376    2033912      71920     353564     980252
-/+ buffers/cache:     671560    3367728
Swap:      7811068          0    7811068

2) -h参数
sunnogo@a3e420:~$ free -h
             total       used       free     shared    buffers     cached
Mem:          3.9G       1.9G       1.9G        70M       345M       957M
-/+ buffers/cache:       655M       3.2G
Swap:         7.4G         0B       7.4G

3) 持续输出，这里的样例输出两次直接手动ctrl + c退出。
sunnogo@a3e420:~$free -hs 3
             total       used       free     shared    buffers     cached
Mem:          3.9G       1.9G       1.9G        70M       345M       957M
-/+ buffers/cache:       655M       3.2G
Swap:         7.4G         0B       7.4G

             total       used       free     shared    buffers     cached
Mem:          3.9G       1.9G       1.9G        70M       345M       957M
-/+ buffers/cache:       655M       3.2G
Swap:         7.4G         0B       7.4G

^C

4) 带输出次数的持续输出。
sunnogo@a3e420:~$ free -hs 3 -c 2
             total       used       free     shared    buffers     cached
Mem:          3.9G       1.9G       1.9G        70M       345M       957M
-/+ buffers/cache:       655M       3.2G
Swap:         7.4G         0B       7.4G

             total       used       free     shared    buffers     cached
Mem:          3.9G       1.9G       1.9G        70M       345M       957M
-/+ buffers/cache:       656M       3.2G
Swap:         7.4G         0B       7.4G
sunnogo@a3e420:~/github/hexo/source/_posts/shell$ 
```

## 命令含义
`free`命令显示结果的含义，该命令读自`/proc/meminfo`，这里只简解free命令用到的内容，待后续再进行补充说明。

* total，即所有可用内存，值为物理内存扣掉一些保留的比特位（多少？）及解压后放到内存中的内核。
* used（“Mem:”行），扣除空闲内存（free）后的内存容量。
* free（“Mem:”行），LowFree和HighFree之和，系统当前未使用的内存总量。
  * HighFree，该区域不是直接映射到内核空间。内核必须使用不同的手法使用该段内存。(Starting with Linux 2.6.19, CONFIG_HIGHMEM is required.)  Total amount of highmem.  Highmem  is  all  memory above  ~860MB  of  physical memory.  Highmem areas are for use by user-space programs, or for the page cache. The kernel must use tricks to access this memory, making it slower to access than lowmem.
  * LowFree，低位可以达到高位内存一样的作用，而且它还能够被内核用来记录一些自己的数据结构。(Starting with Linux 2.6.19, CONFIG_HIGHMEM is required.)  Total amount of lowmem.  Lowmem  is  memory  which can  be  used  for everything that highmem can be used for, but it is also available for the kernel's use for its own data structures.  Among many other things, it is where everything from Slab is allocated. Bad things happen when you're out of lowmem.
* shared，仅共享内存？还是所有进程间通信资源的总和？
* buffers，用来给文件做缓冲大小。 Relatively temporary storage for raw disk blocks that shouldn't get tremendously large (20MB or so).
* cached，被高速缓冲存储器（cache memory）用的内存的大小（等于diskcache minus SwapCache）.
* swap，swap分区的使用情况，swap分区在磁盘分区时确定。被高速缓冲存储器（cache memory）用的交换空间的大小已经被交换出来的内存，但仍然被存放在swapfile中。用来在需要的时候很快被替换而不需要再次打开I/O端口。Memory that once was swapped out, is swapped back in but still also is in the swap file.  (If memory pressure is  high,  these  pages  don't  need to be swapped out again because they are already in the swap file.  This saves I/O.)。
* used（“-/+ buffers/cache:”行），used[2] = used[1] - buffers - cached
* free（“-/+ buffers/cache:”行），free[2] = free[1] + buffers + cached

