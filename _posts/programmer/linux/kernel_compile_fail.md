title: Linux kernel 编译问题记录
date: 2017-01-10 17:31:00
toc: true
tags: [kernel, compile]
categories: kernel
keywords: [Linux, kernel, compile, perl, timeconst.pl, failed, 内核, 编译, 失败, line 373]
description: Linux 编译问题汇总记录。
---

## Can't use 'defined(@array)'

Linux kernel 无法编译通过，报了如下错误：

```
Can't use 'defined(@array)' (Maybe you should just omit the defined()?) at kernel/timeconst.pl line 373. 
```

`kernel/timeconst.pl` 中的代码：

```
372     @val = @{$canned_values{$hz}};                                                                  
373     if (!defined(@val)) {                                                                                    
374         @val = compute_values($hz);                                                                 
375     }                                                                                               
376     output($hz, @val); 
```

将`if (!defined(@val))`改为`if (!@val)`，再次编译就 OK。

原因：perl版本升级到 v5.22.1，发现官网因为一个bug，将defined(@array)去掉了。可以直接使用数组判断非空。

来自：http://blog.5ibc.net/p/48570.html

## error: implicit declaration of functio	'ioread8'

MIPS 架构没有问题的内核模块代码，切到 x86-64，编译时提示如下错误。
通过到 [linux 源代码](http://elixir.free-electrons.com/linux/v2.6.32.38) 中搜索对应函数，发现函数位于 `include/asm-generic/io.h`。
由于涉及的文件比较多，直接通过 sed 对所有的 .c 文件添加该头文件引用，`find . -name "*.c" -exec sed -i '1 i#include <asm-generic/io.h>' "{}" \;`

```
error: implicit declaration of function 'ioread8'                                                      
error: implicit declaration of function 'ioread16be'                                                   
error: implicit declaration of function 'ioread32be'                                                   
error: implicit declaration of function 'iowrite8'                                                     
error: implicit declaration of function 'iowrite16be'                                                  
error: implicit declaration of function 'iowrite32be'                                                  
error: implicit declaration of function 'ioremap'                                                      
error: implicit declaration of function 'iounmap'
```
