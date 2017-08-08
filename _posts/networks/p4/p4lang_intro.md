title: p4 语言简介
date: 2017-08-08 17:34:20
toc: true
tags: [p4]
categories: p4
keywords: [p4, p4lang, introduction, 简介, spec]
description: P4 语言简介
------------------

## 简述

P4 语言为一种领域语言，用于网络转发平面（数据平面）报文处理流水线的描述。

P4 是一种类 C 的专用语言，用于描述数据平面转发流水线。P4 语言源自 SIGCOMM 2014 CCR 论文“[Programming Protocol-Independent Packet Processors](http://www.sigcomm.org/node/3503)”，P4 就是这几个单词的缩写。

简单地，可将 P4 语言与 C 语言进行对比。
* C 语言程序代码  -- gcc 或其他编译器 --> 可执行文件，运行在 x86 CPU / ARM / 51单片机等目标上。
* P4 语言代码 -- P4 编译器--> 固件或其他形式输出，运行在通过 CPU / NPU / FPGA / ASIC 等目标上。

P4 语言的目标：

* 设备无关。P4 语言被设计成与具体实现无关，可以编译到不同的执行机器（亦称为 P4 targets），比如通用 CPU、FPGA、ASIC、NPU 等。厂家提供 P4 target 时，必须同时提供一个后端编译器，将 P4 源码映射到目标交换机模型。
* 协议无关。这意味着 P4 语言不会原生支持任何协议，即便是通用协议，比如 Ethernet、IP、TCP、VxLAN 等。P4 语言描述报文头部格式以及程序中需要的协议字段。
* 可重配置。协议独立性和抽象语言模型允许可重新配置，P4目标应该能够在部署后更改其处理数据包的方式（可能多次）。 这种能力传统上只有通用 CPU 或 NPU 支持，而固定功能 ASIC 不支持。

## P4 语言抽象

## P4 语言规格
目前存在两种并行的语言规格，包含 P4-14 和 P4-16 两种，详见 [p4-spec](https://p4lang.github.io/p4-spec/)。P4-16 相比 P4-14 的改进在于，

## P4 语言代码样例
P4 样例代码，[simple router](https://github.com/p4lang/p4factory/tree/master/targets/simple_router/p4src)。

## 其他网络编程语言
目前常见开放的数据平面编程语言除 P4 外，还有 PoF，不过后者自 2014 年来已基本无更新。二者的对比详见文章 [P4和POF的对比](http://www.sdnlab.com/17869.html)。
