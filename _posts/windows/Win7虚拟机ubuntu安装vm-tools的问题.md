title: 【解决】Win7虚拟机ubuntu安装vm-tools的问题
date: 2015-07-02 22:22:19
toc: false
tags: [windows, 虚拟机]
categories: windows
keywords: [windows, vmplayer, ubuntu]
---

win7 vmplayer中安装ubuntu，在安装vm-tools的时候出现找不到“the location of the directory of C header files that match your running kernel?”。

一般kernel header files安装在``/usr/src/`uname -r`/include``。没有的话自己``apt-get install linux-header-`uname -r` ``。不过发现有了之后还是会出现前面的问题。查了下，发现原因是旧版本的vm-tools会去`linux/version.h`中找一个宏定义`UTS_VERSION`，以前这个宏定义放在`linux/version.h`，但是现在放在`generated/utsrelease.h`，把这个宏定义拷过去就好了。

有可能会说找不到`linux/autoconf.h`，也同样在generated目录中，拷出来就可以了。
