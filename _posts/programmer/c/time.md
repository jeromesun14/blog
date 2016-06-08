title: Linux C 语言时间相关处理
date: 2016-04-23 19:23:30
toc: true
tags: [Linux, c]
categories: c
keywords: [Linux, c, time, clock]
description: Linux c 时间相关处理。
---

* time
* gettimeofday
* jeffies
* clock_gettime，多种类型
* 定时器
* 信号
* 条件锁

<!--more-->

## time()

以下是 `man 2 time`：

```
SYNOPSIS
       #include <time.h>

       time_t time(time_t *t);

DESCRIPTION
       time()  returns  the  time  as  the  number of seconds since the Epoch,
       1970-01-01 00:00:00 +0000 (UTC).

       If t is non-NULL, the return value is also stored in the memory pointed
       to by t.

RETURN VALUE
       On  success,  the value of time in seconds since the Epoch is returned.
       On error, ((time_t) -1) is returned, and errno is set appropriately.

ERRORS
       EFAULT t points outside your accessible address space.

CONFORMING TO
       SVr4, 4.3BSD, C89, C99, POSIX.1-2001.  POSIX does not specify any error
       conditions.
```
