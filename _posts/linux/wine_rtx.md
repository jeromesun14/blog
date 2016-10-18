title: Linux 下使用 RTX
date: 2016-05-22 12:30:24
toc: true
tags: [ubuntu, Linux, wine]
categories: linux
keywords: [Linux, Ubuntu, 16.04, wine, RTX, 2015，字体，截图]
description: Ubuntu 16.04 下 wine rtx 2015，字体正常，截图正常。
---

本文记录 Ubuntu 16.04 64 位下 wine RTX 的过程。最终验证 wine 后的 RTX 字体 OK 无方块，截图正常，可发送截图、文件。

什么是 RTX
--------------
简单的，RTX 是 QQ 的企业版，亦有人称之为 BQQ。如果没有听过这个名称，这篇文章亦可做为 Linux 下 Wine 其他 windows 工具的参考。

> [腾讯通 RTX](http://rtx.tencent.com/rtx/index.shtml)（Real Time eXchange）是腾讯公司推出的企业级即时通信平台。企业员工可以轻松地通过服务器所配置的组织架构查找需要进行通讯的人员，并采用丰富的沟通方式进行实时沟通。文本消息、文件传输、直接语音会话或者视频的形式满足不同办公环境下的沟通需求。

安装最新版 wine
---------------------
详见 [Ubuntu wine wiki](https://wiki.ubuntu.org.cn/Wine)。

ubuntu 官方有自带 wine ，但是推荐用 winehq 官方提供的最新版本 wine ，新版本解决很多以前旧版本的问题。

PPA地址：https://launchpad.net/~wine/+archive/ubuntu/wine-builds

```
sudo add-apt-repository ppa:wine/wine-builds
sudo apt-get update
sudo apt-get install wine-devel
```

要注意，若 `apt-get install wine` 安装的是稳定版（版本一般比较旧）；若 `apt-get install wine-devel` 则安装的是较新的开发版本  ，开发版本经常有不少优化和修正。

如果愿意安装比 devel 稍微更 devel 的 staging 版本的话，可试试 wine-staging。

下载最新 winetricks
-------------------
最新的 winetricks 解决许多下载失败等问题，直接从 github 上下载。[链接](https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks)。

winetricks 下载慢或则失败，可按 winetricks 脚本中写的文件名到 google 搜索、寻找其他下载源。若出现下载的程序版本与 winetricks 要求的版本不一致导致 sha 检测不通过，可通过改 winetricks 中的检测值解决。

安装依赖
--------
为防止 32 位、64 位可能出现不兼容，执行命令的时候配置 `WINEARCH` 为 win32。

```
WINEARCH=win32 WINEPREFIX=~/.wine winetricks msxml3 gdiplus riched20 riched30 ie6 vcrun6 vcrun2005sp1 allfonts
```

安装 rtx2015
---------------

[rtx 官方下载页面](http://rtx.tencent.com/rtx/download/index.shtml)。

```
WINEARCH=win32 WINEPREFIX=~/.wine wine Downloads/rtxclient2015formal.exe
```

安装完后，直接可用。字体、截图及发送、文件发送皆正常。不会出现离线后再上线就登不上的问题。

问题
-----------
* 为防止字体的方块问题，建议将系统的默认语言调整为汉语。如果默认为英语，则某些地方的汉字会显示成方块；
* 不清楚是 WINE 的问题还是 RTX 窗口开太多，RTX 的 CPU 常维持在 20% 左右；
* RTX 可能出现假死，即导致整个 Unity 界面挂住，过一小会儿后才 OK。据说在 Xfce 界面中也会出现此问题。
* RTX 运行久了之后，可能出现系统状态栏的图标点不了。
* RTX 可能在运行过程中出现中文输入法无法调出。本文用的输入法是 fcitx。

截图
-----

* RTX 界面与菜单

界面：
![界面.png](http://upload-images.jianshu.io/upload_images/74971-ecb5c0dfe925f593.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

菜单：
![菜单.png](http://upload-images.jianshu.io/upload_images/74971-d468b7346f145b4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 聊天界面

界面1：
![界面1.png](http://upload-images.jianshu.io/upload_images/74971-e2b5f52aa789ad4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

界面2：（含图片发送）
![界面2.png](http://upload-images.jianshu.io/upload_images/74971-2824058a45efc334.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 消息管理器之会话消息

![消息管理器之会话消息.png](http://upload-images.jianshu.io/upload_images/74971-ab956668a2a322c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 消息管理器之多人会话（含图片）

![消息管理器之多人会话（含图片）.png](http://upload-images.jianshu.io/upload_images/74971-3cbcd477fb304510.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 消息管理器之文件接收

![消息管理器之文件接收.png](http://upload-images.jianshu.io/upload_images/74971-cca92cdb6b1fdb40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 消息管理器之导出会话
将会话导出为 RTF 文件，以下为在 word 中打开的 RTF 文件截图。

![消息管理器之导出会话.png](http://upload-images.jianshu.io/upload_images/74971-5b65965e120748be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* RTX 设置

![RTX 设置.png](http://upload-images.jianshu.io/upload_images/74971-7ef1bdc47f9012cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* RTX 退出界面
注意，如果系统默认语言不是中文，退出界面将的中文显示为方块。

![RTX 退出界面.png](http://upload-images.jianshu.io/upload_images/74971-d53055a7e3c18cf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

