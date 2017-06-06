
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

## 代码

```c
const char data_mem[] = "VmRSS:";
 
/* 打印当前消耗内存 */
void get_mem(int pid, char *func_name)
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
get_mem(getpid(), your_func_name);
```
