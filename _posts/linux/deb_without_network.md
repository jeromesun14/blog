title: 无网络下安装常用 deb 包
date: 2018-02-06 14:08:01
toc: true
tags: [ubuntu, deb]
categories: linux
keywords: [ubuntu, linux, deb, package, without network, 无网络, 常用, common, iptables, gdb, ssh]
description: 无网络下安装常用 deb 包。
---

安装方法：到官网下载对应软件 ubuntu / debian 的 deb 包。

例如 openssh-server，以 ubuntu 16.04 为例，[下载路径](https://packages.ubuntu.com/xenial/openssh-server)，在页面底部，选择对应的硬件架构（arch），比如 amd64。

## openssh-server

安装顺序：

1. openssh-client
2. openssh-sftp-server
3. openssh-server

在 ubuntu 16.04 上验证过。

## gdb

安装顺序：

1. libpython2.7
2. gdb

在 debian jessie 上验证过。

## iptables

安装顺序：

1. libnfnetlink0
2. libxtables10，新版本可能是 libxtables11
3. iptables

在 debian jessie 上验证过。
