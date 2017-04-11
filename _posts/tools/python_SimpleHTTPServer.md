title: 一个灰常灰常简单的 HTTP server
date: 2017-04-11 11:34:20
toc: false
tags: [tools, server]
categories: tools
keywords: [http server, python, simpleHTTPServer]
description: 一个灰常灰常简单的 HTTP server，一个命令部署
---

目标 release 一个版本，该版本含一个 800MB 左右的 vagrant box。思前想后都觉得不方便，因此想搭个 http server，仅供公司内部使用，类似 hfs。然后在[这里](https://superuser.com/questions/43618/which-lightweight-http-or-ftp-server-is-good-for-simple-file-transfer)看到了最佳解决方案 Python SimpleHTTPServer。

使用方式：

* 选择一个目录做为根目录，在该目录下执行 `python -m SimpleHTTPServer [端口号]`，端口号默认为 8000。
* 在浏览器上输入 `http://192.168.204.167:8000`（将 8000 替换为所使用的端口号），就可以在网页上进行操作了。

如果你的目录下有一个叫 index.html 的文件名的文件，那么这个文件就会成为一个默认页，如果没有这个文件，那么，目录列表就会显示出来。

这里以目录列表为例：

```
sunyongfeng@ubuntu:~/pyhttpfs$ tree
.
└── box
    └── 0.9.0
        └── origin32.box
2 directories, 1 files

sunyongfeng@ubuntu:~/pyhttpfs$ python -m SimpleHTTPServer 20090
Serving HTTP on 0.0.0.0 port 20090 ...
```

在浏览器上：

![SimpleHTTPServer](/images/tools/SimpleHTTPServer.png)

coolshell 也写过一篇类似的文章：[非常简单的PYTHON HTTP服务](http://coolshell.cn/articles/1480.html)
