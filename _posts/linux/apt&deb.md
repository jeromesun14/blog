title: apt&deb 工具使用记录
date: 2018-01-15 15:23:00
toc: true
tags: [ubuntu, deb]
categories: linux
keywords: [ubuntu, linux, deb, package, apt, 查看内容]
description: apt 和 deb 相关工具使用记录。
---

## 查看 deb 包内容

* 查看 .deb 文件包含的内容，`dpkg-deb -c packageName.deb`
* 查看已安装 deb 包的内容，`dpkg -L packageName`
