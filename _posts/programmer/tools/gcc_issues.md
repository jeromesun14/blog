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

### 库可以被找到，但是编译时还是提示 `undefined reference to 'Py_Initialize'`

[stackoverflow 链接](http://stackoverflow.com/questions/13951166/undefined-reference-despite-lib-being-found-by-linker)。

* 问题：编译 `gcc -lpython2.7 $(BUILD_CFLAGS) -o $(ELF) $^ -lxxx`，libxxx.a 使用 libpython2.7.so，编译时提示找不到 python 的 symbol。
* 原因：-lxxx 写在 -lpython2.7 之后。

> the linker doesn't yet know that Py_Initialize is a required symbol when it loads libpython2.7.a, so it tosses it away. And then it gets to p.o and throws a fit about the missing symbol. Ordering it this way will let the linker look for the missing symbol in subsequent inputs.
> 
> See: http://gcc.gnu.org/onlinedocs/gcc/Link-Options.html
> 
> It makes a difference where in the command you write this option; the linker searches and processes libraries and object files in the order they are specified. Thus, foo.o -lz bar.o' searches libraryz' after file foo.o but before bar.o. If bar.o refers to functions in `z', those functions may not be loaded.

* 解决：把 -lpython2.7 写在 -lxxx 之后。像这种库中依赖库的地方需要注意。

### relocation R_X86_64_32 against `xxx' can not be used when making a shared object; recompile with -fPIC sub.o: could not read symbols: Bad value

```
x86_64-unknown-linux-gnu-gcc -fPIC -shared -D_LITTLE_ENDIAN -g  -Wno-error=deprecated-declarations  -Wall -D_GNU_SOURCE -lpthread
/home/sunyongfeng/workshop/../build/sub.o /home/sunyongfeng/workshop/../build/undef.o /home/sunyongfeng/workshop/../build/libxy.so
/home/sunyongfeng/workshop/toolchain/toolchain-x86_64/x86_64/bin/../lib/gcc/x86_64-unknown-linux-gnu/4.2.4/../../../../x86_64-unknown-linux-gnu/bin/ld: /home/sunyongfeng/workshop/../build/libproxy/policy/sub.o: relocation R_X86_64_32 against `g_lock' can not be used when making a shared object; recompile with -fPIC
/home/sunyongfeng/workshop/../build/libproxy/policy/sub.o: could not read symbols: Bad value
collect2: ld returned 1 exit status
makefile:69: recipe for target 'build' failed
```
源代码编译也添加 `-fPIC` 选项，在 ARM / MIPS 平台都没此问题，在 x86 才有。
