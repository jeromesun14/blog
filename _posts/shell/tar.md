title: Linux命令行压缩 & 解压缩 - tar
date: 2017-04-17 14:20:18
toc: true
tags: [Linux, 文件系统]
categories: shell
keywords: [linux, command, tar, gzip, bzip2, unzip, unrar, rar, zip]
description: Linux命令行浏览文件内容。
---

tar — The GNU version of the tar archiving utility

* 打包，后缀为 `.tar`，称为 tarfile
* 打包 + 压缩，后缀为 `.tar.gz`、`.tar.bz2`、`.tar.xz`，称为 tarball。此为 linux 最常用的压缩、解压缩方式。

tar 打包命令目前在标准 Linux 发行版上已经整合进压缩/解压缩工具 gzip、bzip2 和 xz。

windows 中常见的压缩包有 `.zip` 和 `.rar`，可通过 `unzip` 和 `unrar` 两个工具进行解压。通过 apt-get install 安装即可。

* 压缩
  + gzip，`tar -zcvf xxx.tar.gz ./*`
  + bzip2，`tar -jcvf xxx.tar.bz2 ./*`
  + xz，`tar -Jcvf xxx.tar.xz ./*`
* 解压缩，`tar -xvf xxx.tar.gz`
* 查看压缩包内容，`tar -tf xxx.tar.gz`

以 [p4factory](https://github.com/p4lang/p4factory) 为例，log 如下。

压缩前大小：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory$ du -hd 0
7.0M    .
```

压缩：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory$ tar -zcvf p4factory.tar.gz ./*
./apps/
./apps/int/
./apps/int/monitor/
./apps/int/monitor/preprocessor.py
./apps/int/monitor/monitor.py
./apps/int/monitor/client_msg_handler.py
./apps/int/monitor/topology.json
./apps/int/monitor/client/
./apps/int/monitor/client/index.html
./apps/int/monitor/client/styles/
./apps/int/monitor/client/styles/main.css
./apps/int/monitor/client/lib/
./apps/int/monitor/client/lib/cola.v1.min.js
...
```

压缩后大小：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory$ ls -alh p4factory.tar.gz 
-rw-rw-r-- 1 sunyongfeng sunyongfeng 771K 4月  17 14:26 p4factory.tar.gz
```

解压过程：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test-tar$ tar xvf ../p4factory/p4factory.tar.gz 
./apps/
./apps/int/
./apps/int/monitor/
./apps/int/monitor/preprocessor.py
./apps/int/monitor/monitor.py
./apps/int/monitor/client_msg_handler.py
./apps/int/monitor/topology.json
./apps/int/monitor/client/
./apps/int/monitor/client/index.html
./apps/int/monitor/client/styles/
./apps/int/monitor/client/styles/main.css
./apps/int/monitor/client/lib/
./apps/int/monitor/client/lib/cola.v1.min.js
...
```
