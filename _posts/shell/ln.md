title: Linux命令行创建“快捷方式” - ln
date: 2014-02-21 00:03:59
tags: [Linux, 文件系统]
categories: shell
toc: true
keywords: [linux, command, ln, file system]
description: Linux命令行删除普通文件、目录文件。
---

将ln命令说成创建“快捷方式”只是用于比喻，实际上ln命令与Linux文件系统的特性悉悉相关。

# 基础知识简述
文件都有**文件名**与**数据**，数据在Linux上被分成两个部分：用户数据（user data）与元数据（metadata）。**用户数据**，即文件数据块（data block），数据块是记录文件真实内容的地方；而**元数据**则是文件的附加属性，如文件大小、创建时间、所有者等信息。在Linux中，元数据中的inode号（inode 是文件元数据的一部分但其并不包含文件名，inode号即索引节点号）才是文件的唯一标识而非文件名。文件名仅是为了方便人们的记忆和使用，系统或程序通过文件名索引到inode号再去寻找正确的文件数据块。

为解决文件的共享使用，Linux系统引入了两种链接：硬链接（hard link）与软链接（又称符号链接，即soft link或symbolic link）。链接为Linux 系统解决了文件的共享使用，还带来了隐藏文件路径、增加权限安全及节省存储等好处。

<!--more-->

* 若一个inode号对应多个文件名，则称这些文件为**硬链接**。换言之，硬链接就是同一个文件使用了多个别名。创建一个硬链接之后，inode不变，但是硬连接总数（也称为引用计数）有增加，当删除文件时，只有硬连接总数为0时，系统才会真正删除文件。
* 软链接与硬链接不同，若文件用户数据块中存放的内容是另一文件的路径名的指向，则该文件就是**软连接**。软链接就是一个普通文件，只是数据块内容有点特殊。软链接有着自己的inode号以及用户数据块，即有新文件生成，这个新文件（如上symbol.sh）指向旧文件（if.sh）。这个有点像Windows中的快捷方式。快捷方式指向的文件被删除了，则快捷方式无效；如果把快捷方式删除了，也不影响原来的文件。

（以上关于软、硬链接的说明摘抄自IBM文档[《理解 Linux 的硬链接与软链接》](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/)）

# ln命令说明
ln - make links between files  
常用格式：ln [OPTION] TARGET LINK_NAME  
`ln target link_name`等同于`cp -l target link_name`  
`ln -s target link_name`等同于`cp -s target link_name`  
`-f`，覆盖已有的link。

> TIPs: 
> * [Linux文件复制命令 - cp](http://sunnogo.tk/201402/shell/cp.html)也有创建链接的功能。使用`-s`表示创建符号链接，`-l`表示创建硬链接。
> * **注意**：windows默认文件系统不支持软链接，如果将含软链接的压缩包在windows文件系统中，软链接将失效。

```
sunnogo@a3e420:~/test/script$ ln if.sh lnlink.sh
sunnogo@a3e420:~/test/script$ ls -li
total 16
7340612 -rwxrwxr-x 3 sunnogo sunnogo  94  6月 21  2013 if.sh
7340612 -rwxrwxr-x 3 sunnogo sunnogo  94  6月 21  2013 link.sh
7340612 -rwxrwxr-x 3 sunnogo sunnogo  94  6月 21  2013 lnlink.sh
7340899 -rw-rw-r-- 1 sunnogo sunnogo 519 11月 12 00:33 prof.c
7340881 lrwxrwxrwx 1 sunnogo sunnogo   5  2月 23 21:16 symbol.sh -> if.sh
sunnogo@a3e420:~/test/script$ ln -s if.sh lnsymbol.sh
sunnogo@a3e420:~/test/script$ ls -li
total 16
7340612 -rwxrwxr-x 3 sunnogo sunnogo  94  6月 21  2013 if.sh
7340612 -rwxrwxr-x 3 sunnogo sunnogo  94  6月 21  2013 link.sh
7340612 -rwxrwxr-x 3 sunnogo sunnogo  94  6月 21  2013 lnlink.sh
7340895 lrwxrwxrwx 1 sunnogo sunnogo   5  2月 23 21:26 lnsymbol.sh -> if.sh
7340899 -rw-rw-r-- 1 sunnogo sunnogo 519 11月 12 00:33 prof.c
7340881 lrwxrwxrwx 1 sunnogo sunnogo   5  2月 23 21:16 symbol.sh -> if.sh
```

> Written with [StackEdit](https://stackedit.io/).
