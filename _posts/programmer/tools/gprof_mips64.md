title: octeon mips64 上使用 gprof 记录
date: 2017-06-08 23:34:00
toc: true
tags: [profiling]
categories: programmer
keywords: [linux, gcc, pg, gprof, multi thread, kill, SIGUSR1, mips64, gmon, empty]
description: octeon mips64 CPU 上使用 gprof 记录。
---

## 为何使用 gprof

对 mips64 设备上某业务进行性能调优，由于 Linux 系统为深度定制系统且 mips CPU 的限制，valgrind / 内核 perf 工具都有法使用（尚未研究 OProfile）。strace 只能查看
