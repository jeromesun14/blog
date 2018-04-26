title: Linux下载命令 - wget
date: 2018-04-25 17:31:00
toc: true
tags: [Linux]
categories: shell
keywords: [linux, command, wget]
description: Linux 下载命令 wget
---

常用选项：

* -r  : 遍历所有子目录
* -np  : 不到上一层子目录去
* -nH  : 不要将文件保存到主机名文件夹
* -R index.html, 不下载 index.html 文件
* -o logfile, 即 --output-file=logfile, 保存 log 到 logfile
* -A acclist, 即 --accept acclist, 下载的白名单，例如，-A deb,log,gz,ko，表示下载后续为 .deb / .log / .gz / .ko 的文件。如果使用通配符，则不再视为后缀
* -R rejlist, 下载的黑名单
* --no-use-server-timestamps, 
* -l depth, 即 --level=depth，递归下载的深度
