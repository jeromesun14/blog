title: Linux 链接库使用记录
date: 2016-05-25 20:03:24
toc: true
tags: [Linux, lib]
categories: programmer
keywords: [linux, c, gcc, lib]
description: 记录 Linux 静态、动态链接库的一些常用使用方法。
---

本文记录 Linux 静态、动态链接库的一些常用使用方法。

## 查看符号

* `nm` 命令，`nm -A libxxx.so 2>/dev/null | grep "your_symbol"`

