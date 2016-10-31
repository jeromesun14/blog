title: makefile 学习记录
date: 2016-04-15 20:03:24
toc: true
tags: [makefile]
categories: programmer
keywords: [makefile, compile, gcc, make]
description: makefile 使用备忘
---

## miscellaneous

### 判断 32 位还是 64 位

来自 [stackoverflow](http://stackoverflow.com/questions/4096173/how-do-i-create-a-single-makefile-for-both-32-and-64-bit) 的答案:

```
ARCH := $(shell getconf LONG_BIT)

CPP_FLAGS_32 := -D32_BIT ...  Some 32 specific compiler flags ...
CPP_FLAGS_64 := -D64_BIT

CPP_FLAGS := $(CPP_FLAGS_$(ARCH))  ... all the other flags ...
```

### 清除某个 CFLAGS 编译选项

来自 [stackoverflow](http://stackoverflow.com/questions/17316426/make-override-a-flag) 的答案：

```
CFLAGS := $(filter-out -Werror,$(CFLAGS))
```

### 打印 64 bits

http://stackoverflow.com/questions/9225567/how-to-print-a-int64-t-type-in-c

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
