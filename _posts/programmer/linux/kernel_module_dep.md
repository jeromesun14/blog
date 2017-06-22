title: 查看 linux kernel module 依赖关系
date: 2017-06-22 16:58:00
toc: true
tags: [kernel]
categories: kernel
keywords: [Linux, kernel, module, dependency, 依赖, 关系, 内核, 模块]
description: linux 内核模块依赖关系查看方法汇总。
---

问题背景：从专有系统移植某个特定功能的内核模块到 Linux 发行版，确认该模块的依赖关系。

http://xmodulo.com/how-to-check-kernel-module-dependencies-on-linux.html

* lsmod
* /lib/modules/2.6.32.13-Cavium-Octeon/modules.dep
* modprobe --show-dependency
* depmod
