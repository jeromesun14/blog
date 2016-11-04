title: Linux 编译时 configure 问题记录
date: 2016-11-04 10:13:24
toc: true
tags: [compile, configure]
categories: programmer
keywords: [linux, configure, makefile, build, host, target]
description: 编译过程中 configure 文件相关的问题记录和学习笔记
---

--host, --target and --build 选项
=================================

这部分内容译自：[Configure with --host, --target and --build options](http://jingfenghanmax.blogspot.co.uk/2010/09/configure-with-host-target-and-build.html)。

在交叉编译时，很可能涉及到这三个容易混淆的选项：
* `--build`，程序在哪个系统上编译；
* `--host`，编译生成的程序在哪个系统上运行；
* `--target`，只在交叉编译工具链上使用；交叉编译工具链编译的程序在哪个目标系统上运行。

以 tslib （一个鼠标驱动库）为例：这个动态库在 x86 linux PC 上编译、构建，生成的结果在 arm linux 系统上运行。
```
./configure --host=arm-linux --build=i686-pc-linux-gnu
```

以 gcc 为例：（例子有点绕，但是很到位）
* `--build`，在 x86 linux PC 上编译 gcc；
* `--host`，编译生成的 gcc 可在嵌入式 arm linux 系统上运行；
* `--target`，在 arm linux 上 运行 gcc，可编译、构建生成二进制可执行文件，这些可执行文件最终将运行在 x86 linux 系统上。

```
./configure --target=i686-pc-linux-gnu --host=arm-linux --build=i686-pc-linux-gnu
```

