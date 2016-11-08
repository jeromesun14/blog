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

### 快速解决 -Werror 选项出错
带 `-Werror` 选项，一有 warning 就出错，有时不同架构间移值代码很容易出现。
```
cc1: warnings being treated as errors
```
快速解决的方法：添加选项 `-Wno-error`。

### -Werror 选项不生效
`-Wno-error` 一定要在 `-Werror` 之后，从实际运行看，放在后面的 option 生效。
源代码：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test$ cat abc.c
int main()
{
    printf("sv\n");
}
```
正常编译：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test$ gcc -o abc abc.c
abc.c: In function ‘main’:
abc.c:3:5: warning: implicit declaration of function ‘printf’ [-Wimplicit-function-declaration]
     printf("sv\n");
     ^
abc.c:3:5: warning: incompatible implicit declaration of built-in function ‘printf’
abc.c:3:5: note: include ‘<stdio.h>’ or provide a declaration of ‘printf’
```
如果 `-Werror` 在 `-Wno-errno` 之后，则 warning 还是当 error。
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test$ gcc -o abc abc.c -Wno-error  -Werror
abc.c: In function ‘main’:
abc.c:3:5: error: implicit declaration of function ‘printf’ [-Werror=implicit-function-declaration]
     printf("sv\n");
     ^
abc.c:3:5: error: incompatible implicit declaration of built-in function ‘printf’ [-Werror]
abc.c:3:5: note: include ‘<stdio.h>’ or provide a declaration of ‘printf’
cc1: all warnings being treated as errors
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test$ gcc -o abc abc.c -Wno-error  -Werror -Wno-error
abc.c: In function ‘main’:
abc.c:3:5: warning: implicit declaration of function ‘printf’ [-Wimplicit-function-declaration]
     printf("sv\n");
     ^
abc.c:3:5: warning: incompatible implicit declaration of built-in function ‘printf’
abc.c:3:5: note: include ‘<stdio.h>’ or provide a declaration of ‘printf’
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test$ 
```

