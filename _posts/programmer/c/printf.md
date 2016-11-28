title: linux c printf 相关记录
date: 2016-11-01 13:03:24
toc: true
tags: [Linux, c]
categories: c
keywords: [Linux, c, printf, 64位]
description: Linux c printf 函数相关记录。
---

打印 64 位 int
--------------
http://stackoverflow.com/questions/9225567/how-to-print-a-int64-t-type-in-c

With C99 the %j length modifier can also be used with the printf family of functions to print values of type int64_t and uint64_t:
```
#include <stdio.h>
#include <stdint.h>

int main(int argc, char *argv[])
{
    int64_t  a = 1LL << 63;
    uint64_t b = 1ULL << 63;

    printf("a=%jd (0x%jx)\n", a, a);
    printf("b=%ju (0x%jx)\n", b, b);

    return 0;
}
```
