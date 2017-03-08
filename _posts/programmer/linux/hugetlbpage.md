title: Huge page 概述 
date: 2017-03-08 13:43:24
toc: true
tags: [Linux]
categories: linux
keywords: [Linux, memory, huge page]
description: Huge Page 概述。
---

参考：

* https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt
* https://wiki.debian.org/Hugepages

> When a process uses some memory, the CPU is marking the RAM as used by that process. For efficiency, the CPU allocate RAM by chunks of 4K bytes (it's the default value on many platforms). Those chunks are named pages. Those pages can be swapped to disk, etc.
> 
> Since the process address space are virtual, the CPU and the operating system have to remember which page belong to which process, and where it is stored. Obviously, the more pages you have, the more time it takes to find where the memory is mapped. When a process uses 1GB of memory, that's 262144 entries to look up (1GB / 4K). If one Page Table Entry consume 8bytes, that's 2MB (262144 * 8) to look-up.
> 
> Most current CPU architectures support bigger pages (so the CPU/OS have less entries to look-up), those are named Huge pages (on Linux), Super Pages (on BSD) or Large Pages (on Windows), but it all the same thing.
