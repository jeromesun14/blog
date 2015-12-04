title: Linux命令行拷贝文件 - cp
date: 2014-02-21 00:03:59
tags: [Linux, 文件系统]
categories: shell
toc: true
keywords: [linux, command, cp, file system]
description: Linux命令行复制普通文件、目录文件。
---

cp - copy files and directories  
命令语法：`cp [option] source destination`  

# 基本参数
cp的参数较多，一般使用`-a`参数即可完成大多数的拷贝操作需求。个人亦喜欢使用`-aux`参数查看文件的更新情况。
`-a`，等于-dpR，归档文件，并保留它们的属性。即整个目录保留属性复制。
`-u`，update，仅在源文件比目标文件新的情况下复制（相当于更新）。  
`-v`，verbose，详细模式，解释到底发生了什么。这个参数在其他命令中也很常见。

<!--more-->

```
sunnogo@a3e420:~/test/script$ cp -auv ../prof.c .
‘../prof.c’ -> ‘./prof.c’
sunnogo@a3e420:~/test/script$ 
```

# 其他常见参数：  
`-f`，force，强制覆盖已存在的目标文件，不提示。这个参数在其他命令中也很常见。
`-l`，hard link，创建文件链接，而非复制。
`-s`，symbolic link，创建一个符号链接而非复制文件。
`-p`，preserve，保存属性信息，如owership、timestamp等。  
`-R`，recursive，递归复制，复制目录需要加上这个选项。
`-x`，仅限于当前文件系统的复制。

```
sunnogo@a3e420:~/test/script$ ls -li
total 4
7340612 -rwxrwxr-x 1 sunnogo sunnogo 94  6月 21  2013 if.sh
sunnogo@a3e420:~/test/script$ 
sunnogo@a3e420:~/test/script$ cp -l if.sh link.sh
sunnogo@a3e420:~/test/script$ ls -li
total 8
7340612 -rwxrwxr-x 2 sunnogo sunnogo 94  6月 21  2013 if.sh
7340612 -rwxrwxr-x 2 sunnogo sunnogo 94  6月 21  2013 link.sh
sunnogo@a3e420:~/test/script$ 
sunnogo@a3e420:~/test/script$ cp -s if.sh symbol.sh
sunnogo@a3e420:~/test/script$ ls -li
total 8
7340612 -rwxrwxr-x 2 sunnogo sunnogo 94  6月 21  2013 if.sh
7340612 -rwxrwxr-x 2 sunnogo sunnogo 94  6月 21  2013 link.sh
7340881 lrwxrwxrwx 1 sunnogo sunnogo  5  2月 23 21:16 symbol.sh -> if.sh
```

注意cp -l和cp -s的差异，体现在ls -li中显示的inode是否一致和文件的硬链接总数。关于软、硬链接，在[《Linux命令行创建“快捷方式” - ln》](http://sunnogo.tk/201402/shell/ln.html)一文描述。

> Written with [StackEdit](https://stackedit.io/).
