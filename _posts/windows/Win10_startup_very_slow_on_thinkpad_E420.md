title: Windows 10 在 thinkpad E420 启机慢
date: 2016-01-18 19:40:18
toc: true
tags: [windows]
categories: windows
keywords: [windows 10, thinkpad, E420, 启机, 慢, 黑屏, 磁盘, 使用率, 高]
description: 本文描述 thinkpad e420 安装 windows 10 后启机慢的问题，及优化解决方法。
---

## 启动慢
### 现象
启动过后，黑屏大概 1 分半钟才进入登陆界面。总的启动时间 2 分钟以上。

### 解决
[win10开机黑屏很久（amd双显卡）](http://jingyan.baidu.com/article/0eb457e52bfeb003f1a905ba.html)，可能是 AMD HD7400 显卡特有问题。
禁用 ULPS （Ultra Low Power State），注册表如下。ULPS 用于节能。

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4D36E968-E325-11CE-BFC1-08002BE10318}\0000]
"EnableULPS"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4D36E968-E325-11CE-BFC1-08002BE10318}\0001]
"EnableULPS"=dword:00000000
```

## 磁盘使用率高
很可能是误报，因为刚装完机可能在后台安装更新。
优化可参照：[Win10磁盘占用率100%的原因及解决方法](http://www.windows10.pro/win10-usage-of-your-disk-100/)
