title: 【解决】thinkpad没有break键 & win7远程桌面全屏与退出
date: 2015-07-02 22:12:18
toc: false
tags: [windows, 远程桌面]
categories: windows
keywords: [thinkpad, windows, break键，远程桌面，全屏]
---
使用win7远程公司的电脑，但是全屏之后看不到中间那个最大化最小化的小条条了，不知道要怎么退出全屏的远程桌面。经查，有几个解决方法：
 
* 最简单粗暴的方法，能过QQ或其他办法让主机弹窗。
* 进入mstsc远程前，配置一下远程桌面的参数。`选项`->`本地资源`->`键盘`->选择`应用Windows组合键`在`这台计算机上`，这样进入远程桌面之后想切出来就可以直接用`alt + ctrl`或者其他各种方式。

<!--more-->

![应用windows组合键](http://blog.chinaunix.net/attachment/201209/20/27786025_1348154811QeO6.png)

* 使用快捷键`ctrl + alt + break`切换远程桌面全屏与退出。问题是，我的笔记本上没有break/pause键！搜了好久，才发现：thinkpad下（e420试验没错），Fn + p = pause, Fn + b = break。
  * 玩dota的同学应该可以用warkey设一个break键出来；
  * 像网上某些蛋疼的回答：搞个外接键盘，标准键盘都有。。

![这里写图片描述](http://blog.chinaunix.net/attachment/201209/20/27786025_13481545875Fm3.png)
另外远程桌面默认不是全屏的，在mstsc远程前，配置`选项`->`显示配置`，把全屏滑动条拉到最右边就可以了。

