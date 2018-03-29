title: Linux时间相关
date: 2018-03-29 15:34:20
toc: true
tags: [Linux, time, clock]
categories: shell
keywords: [linux, command, time, clock, tick, hz]
description: Linux时间相关操作。
---

## 配置系统时间

* 配置硬件时钟，`hwclock --set --date="03/29/2018 15:31"`
* 同步硬件时钟到系统时间，`hwclock --hctosys`
* 同步系统时间到硬件时钟，`hwclock --systohc`
* 查看硬件时钟，`hwclock -r`
* 查看系统时间，`date`

