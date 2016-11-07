title: rpm使用记录
date: 2016-03-16 09:48:18
toc: true
tags: [Linux, rpm]
categories: tools
keywords: [Linux, rpm]
description: 本文记录 rpm 包管理命令常用方法，备忘。
---

列出rpm包的内容：
```
rpm -qpl *.rpm
```

解压rpm包的内容：（没有安装，就像解压tgz包一样rpm包）
```
rpm2cpio *.rpm | cpio -div
```
