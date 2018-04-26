title: xterm / gnome-terminal 无法 tab 补全
date: 2017-04-25 12:43:00
toc: true
tags: [ubuntu, xterm, gnome-terminal]
categories: linux
keywords: [ubuntu, 16.04, linux, xterm, gnome-termianl, tab completion, 无法补全]
description: ubuntu 下 gnome-terminal / xterm tab 无法自动补全问题。
---

## 问题：gnome-terminal / xterm 的 profile 失效，且无法进行 tab 补全。

![xterm tab commpletion not working](/images/linux/term/xterm.ugly.png)

从命令行起 gnome-terminal / xterm，带执行参数 bash 即可。

* `gnome-terminal -x bash`
* `xterm -e bash`

![xterm tab completion works](/images/linux/term/xterm.ok.png)

## 问题：term 太小，vim 打开看到篇幅很小

```
stty rows 32 cols 114
```
