title: vagrant 使用记录
date: 2017-03-31 11:39:24
toc: true
tags: [vagrant, virtualBox]
categories: programmer
keywords: [vagrant, virtualBox, host ping virtualBox]
description: Vagrant + virtualBox 使用记录
---

# vagrant 如何 ping 通虚拟机

## vagrant 默认网络通信机制
vagrant 默认使用 NAT 端口映射，可以通过端口映射访问虚机内部端口号。但是默认情况下，vagrant 可以访问外网，但是外部世界访问不了 vagrant （类似以前在学校，学校内可以访问外网，但是外网访问不了教育网）。
