title: linux 串口相关
date: 2018-03-30 11:19:24
toc: true
tags: [Linux, serialport]
categories: Linux
keywords: [Linux, serial, serial port, ttyS0, ttyUSB0, grub, startup, baudrate, 波特率]
description: linux 串口相关使用汇总
---

## 开机自启动，通过串口输出

grub 的配置文件中修改相应行：

```
GRUB_CMDLINE_LINUX='console=tty0 console=ttyS1,19200n8'
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=19200 --unit=1 --word=8 --parity=no --stop=1"
```

## 运行时修改串口配置

使用 `stty` 命令。详见 `man tty` 描述。

`stty -F /dev/ttyS0 speed 115200`，修改 ttyS0 波特率为 115200

## 共享串口

可使用 ser2net 共享串口，详见之前整理的 [ser2net 用法](http://sunyongfeng.com/201509/linux/dev-conf.html#Ser2net)。
