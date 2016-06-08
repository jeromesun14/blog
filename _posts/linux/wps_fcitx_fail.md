title: fcitx 在 WPS for Linux 中无法使用
date: 2016-05-26 12:30:24
toc: true
tags: [ubuntu, wps, fcitx]
categories: linux
keywords: [wps, wpp, et, Ubuntu, 16.04, linux, fcitx, 无法输入]
description: ubuntu 下，设置环境变量，让 fcitx 在 WPS for Linux 可以正常使用。
---

问题： fcitx 在 WPS for Linux 中调不出来，在 Linux 的其他应用中可以正常使用。
原因：环境变量未正确设置。
解决： Google 搜索，来自 [Linux 教程](http://www.linuxdiyf.com/linux/13396.html)：

在 `/usr/bin/wps` （word）、`/usr/bin/wpp` （powerpoint）、`/usr/bin/et` （excel）脚本的开始，加入 `export XMODIFIERS="@im=fcitx"` 和 `export QT_IM_MODULE="fcitx"`。
以 wps 为例：

```
$ sudo vi /usr/bin/wps
#!/bin/bash

export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"

gOpt=
#gOptExt=-multiply
gTemplateExt=("wpt" "dot" "dotx")
```
