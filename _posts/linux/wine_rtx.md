title: Linux 下使用 RTX
date: 2016-05-22 12:30:24
toc: true
tags: [ubuntu, Linux, wine]
categories: linux
keywords: [Linux, Ubuntu, 16.04, wine, RTX, 2015，字体，截图]
description: Ubuntu 16.04 下 wine rtx 2015，字体正常，截图正常。
---

记录 Ubuntu 16.04 64 位下 wine RTX 的过程。最终字体 OK，无方块，截图正常，可发送截图、文件。

安装最新版 wine
---------------
详见 [Ubuntu wine wiki](https://wiki.ubuntu.org.cn/Wine)。

ubuntu 官方自带了 wine ，但是推荐用 winehq 官方提供的最新版本 wine ，新版本解决了很多以前显得麻烦的问题。

PPA地址：https://launchpad.net/~wine/+archive/ubuntu/wine-builds

```
sudo add-apt-repository ppa:wine/wine-builds
sudo apt-get update
sudo apt-get install wine-devel
```

要注意，若安装 wine 包是老的稳定版，新开发版本是 wine-devel ，经常有不少优化和修正。

如果愿意安装比 devel 稍微更 devel 的 staging 版本的话，可以试试 wine-staging。

下载最新 winetricks
-------------------
最新的 winetricks 解决许多下载失败等问题，直接从 github 上下载。[链接](https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks)。

安装依赖
--------
以防 32 位、64 位可能出现的不兼容问题，执行命令的时候配置 `WINEARCH`。

```
WINEARCH=win32 WINEPREFIX=~/.wine winetricks msxml3 gdiplus riched20 riched30 ie6 vcrun6 vcrun2005sp1 allfonts
```

安装 rtx2015
------------

[rtx 官方下载页面](http://rtx.tencent.com/rtx/download/index.shtml)。

```
WINEARCH=win32 WINEPREFIX=~/.wine wine Downloads/rtxclient2015formal.exe
```

安装完后，直接可用。字体、截图及发送、文件发送皆正常。
