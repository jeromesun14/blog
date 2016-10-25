title: GCC 问题记录
date: 2016-02-28 14:13:20
toc: true
tags: [gcc]
categories: programmer
keywords: [linux, gcc, compile, 编译]
description: Linux c gcc 使用问题记录。
---

## errors

### error: dereferencing pointer to incomplete type
原因：使用 `typedef struct {} xxx_t` 定义类型 `xxx_t`，定义变量时，使用 `struct xxx_t` ，应为 `xxx_t`。

```
typedef struct xxx_s {
    int a;
    int b;
} xxx_t;

int func(void)
{
   struct xxx_t st;
   
   st.a = 1; // <---- 这里会报错。
}
```

### 找不到 libz.so.1

交叉编译时，找不到 libz.so.1，但是 sudo apt-get install zlib1g 相关的所有包都已安装。

```
/home/sunyongfeng/workshop/rgosm-build/.toolchain-arm-cortex_a9/arm-cortex_a9/arm-cortex_a9-linux-gnueabi/bin/../lib/gcc/arm-cortex_a9-linux-gnueabi/4.4.6/../../../../arm-cortex_a9-linux-gnueabi/bin/as: error while loading shared libraries: libz.so.1: cannot open shared object file: No such file or directory
```

原因：交叉编译 target 为 arm 32 bit，host 为 x64。
解决：[来自 stackoverflow](http://stackoverflow.com/questions/21256866/libz-so-1-cannot-open-shared-object-file)，安装 zlib1g:386 版本，`sudo apt-get install zlib1g:i386`


