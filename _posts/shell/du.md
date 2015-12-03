title: Linux命令行查看目录及子目录大小 - du
date: 2014-12-23 23:48:18
toc: true
tags: [Linux命令, 文件系统]
categories: shell
keywords: [linux, command, du]
description: Linux命令行查看目录及子目录大小。
---

在[ls][1]命令中，可通过`ls -alh`查看文件的大小，但是此时看到的目录文件大小是Linux目录文件实际的大小，一般是4KB，而不是一般意义的目录总大小，命令du可用于查看目录的总大小。

```
du - estimate file space usage
     Summarize disk usage of each FILE, recursively for directories.
```

<!--more-->

du命令默认递归把本目录以下的所有目录及文件的大小都输出到控制台。如果各目录层次下的文件太多，会导致控制台输出的内容太多。

一般情况下会使用`-d N`参数或者`--max-depth=N`来控制du命令递归输出的目录层次数。N表示递归输出到第几级目录。`-h`参数是让输出的文件大小值使用Human readable的形式表示，即以KB/MB/GB的形式出现，默认以KB为单位输出。

# 查看本目录总大小
使用0级目录即可。注意与ls -alh的结果对比，ls的结果也有个total总计，但是这个total并不递归累加每个子目录下各个文件的大小，而是把本目录下的子目录当成4KB的普通文件。

```
sunnogo@a3e420:~/github/hexo$ du -d 0 -h
181M    .

sunnogo@a3e420:~/github/hexo$ ls -alh
total 163M
drwxrwxr-x  8 sunnogo sunnogo 4.0K Dec 24 23:09 .
drwxrwxr-x  6 sunnogo sunnogo 4.0K Dec 12 22:41 ..
-rw-rw-r--  1 sunnogo sunnogo 2.0K Dec 20 23:13 _config.yml
-rw-rw-r--  1 sunnogo sunnogo 105K Dec 24 00:49 db.json
-rw-rw-r--  1 sunnogo sunnogo 7.3K Dec 13 19:38 debug.log
drwxrwxr-x 13 sunnogo sunnogo 4.0K Dec 24 00:49 .deploy
-rw-rw-r--  1 sunnogo sunnogo   68 Dec 12 22:41 .gitignore
drwxr-xr-x  5 sunnogo sunnogo 4.0K Dec 12 22:42 node_modules
-rw-rw-r--  1 sunnogo sunnogo  186 Dec 12 22:47 package.json
drwxrwxr-x 12 sunnogo sunnogo 4.0K Dec 20 23:19 public
drwxrwxr-x  2 sunnogo sunnogo 4.0K Dec 12 22:41 scaffolds
drwxrwxr-x  5 sunnogo sunnogo 4.0K Dec 20 22:47 source
drwxrwxr-x  4 sunnogo sunnogo 4.0K Dec 12 22:46 themes
-rw-rw-r--  1 sunnogo sunnogo 163M Dec 24 23:08 veryLargeFile.txt

```

# 查看本目录下各个子目录的总大小
使用一级目录递归输出即可。注意，一级递归输出并没有输出本目录下的普通文件，但是大小已经统计在内。

```
sunnogo@a3e420:~/github/hexo$ du -d 1 -h
2.5M    ./public
20K     ./scaffolds
6.3M    ./node_modules
4.5M    ./themes
32K     ./source
5.2M    ./.deploy
181M    .
```

# 对输出的结果进行排序
简单地，du命令可与[sort][2]命令配合按大小或文件名输出结果。亦可再配合[head][3]或[tail][4]命令只显示排序前几或倒数前几的结果。

```
sunnogo@a3e420:~/github/hexo$ du -d 1 | sort -nr 
185104  .
6376    ./node_modules
5324    ./.deploy
4592    ./themes
2560    ./public
32      ./source
20      ./scaffolds

sunnogo@a3e420:~/github/hexo$ du -d 1 | sort -nr | head -n 3
185104  .
6376    ./node_modules
5324    ./.deploy
```

* sort的`-n`选项表示按数值排序，`-r`表示反向排序即从大到小排序，sort默认使用。
* 注意符号`|`，在linux shell中这是一个很重要的概念，名为管道，即将前道工序输出的结果做为后一道工序的输入。
* head命令显示输入的前n行，tail命令则相反。

# 只输出大小大于某个特定值的文件
通过`-t SIZE`或`--threshold=SIZE`实现本功能，**注意** Size的单位是Byte。
```
sunnogo@a3e420:~/github/hexo$ du -d 1 -h -t 4500000
6.3M    ./node_modules
4.5M    ./themes
5.2M    ./.deploy
181M    .
```

# 忽略统计某些特定类型或匹配文件的大小
通过`-X PATTERN`或`--exclude=PATTERN`实现本功能。注意PATTERN是支持通配符，并不支持正则表达式。

一般用于过滤特定后缀的文件。如果想一次过滤多种PATTERN的文件，可通过多次使用`-X`或`--exclude`实现。

```
sunnogo@a3e420:~/github/hexo$ du -hd 1 --exclude '*.js'
2.3M    ./public
20K     ./scaffolds
2.4M    ./node_modules
4.2M    ./themes
32K     ./source
5.0M    ./.deploy
177M    .
sunnogo@a3e420:~/github/hexo$ du -hd 1 --exclude '*.js' --exclude '*.txt'
2.3M    ./public
20K     ./scaffolds
2.4M    ./node_modules
4.2M    ./themes
32K     ./source
5.0M    ./.deploy
14M     .
```

[1]: http://sunnogo.tk/shell/ls.html
[2]: Not_support_yet
[3]: Not_support_yet
[4]: Not_support_yet

> Written with [StackEdit](https://stackedit.io/).

