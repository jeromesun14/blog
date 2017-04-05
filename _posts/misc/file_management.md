title: 文件归档管理经验
date: 2017-04-05 16:25:00
toc: true
tags: [misc]
categories: misc
keywords: [文件, 归档]
description: 文件归档管理经验。
---

# 归档的文件命名经验
组内学习 SIGCOMM / NSDI 论文，对 SIGCOMM / NSDI 论文进行归档。

* 小组 A 负责 SIGCOMM 论文，下载文章，并按文章名重命名文件、归档。例如，[Dealine-Aware Datacenter TCP](http://conferences.sigcomm.org/sigcomm/2012/paper/sigcomm/p115.pdf)，默认下载后的文件名为 `p115.pdf`
* 小组 B 负责 NSDI 论文，下载文章，不进行重命名，但是通过 excel 表格建立下载链接索引。例如，[OSA: An Optical Switching Architecture for Data Center Networks with Unprecedented Flexibility](https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final88.pdf)，默认下载后文件名为 `nsdi12-final88.pdf`，建立的索引路径为`http://svn.xxx.net/svndoc/d11/2015/一部/技术研究/论文学习/NSDI/2012/Data Center Networking/nsdi12-final88.pdf`

一年后...

部门组织架构调整，原存放论文学习的 URL 为 `http://svn.xxx.net/svndoc/d11/2015/n部/技术研究/论文学习/NSDI/2012/Data Center Networking/nsdi12-final88.pdf`。如果此时想查阅 `OSA: An Optical Switching Architecture for Data Center Networks with Unprecedented Flexibility`，则只能先查找其索引路径，看存储的文件名是什么，然后才能找得到文件。而如果像小组 A 的存档方法，通过 everything 或 listary 等工具，直接输入文件名就可以快速找到文件了。
