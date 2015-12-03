title: Linux命令行剪切、重命名文件 - mv
date: 2014-02-21 00:03:59
tags: [Linux命令, 文件系统]
categories: shell
keywords: [linux, command, mv, file system]
description: Linux命令行剪切、重命名普通文件、目录文件。
---

mv - move (rename) files  
常用方式：

* 重命名，`mv dir1/file1 dir2/file2`，如果已有dir2/file2的存在，则就像cp命令那样，有覆盖提示（-i）、更新（-u）、强制覆盖（-f）等一样的选项。
* 移动单个文件/目录（即剪切），`mv file1 dir1`
* 移动多个文件/目录到同一个目录，`mv -t dest_dir file1 file2...`

<!--more-->

```
1. 创建三个文件
sunnogo@a3e420:~/test/mv$ echo "first file" > file1
<----创建文件的方式有很多种，touch只是其中一种。
sunnogo@a3e420:~/test/mv$ cat file1 
first file
sunnogo@a3e420:~/test/mv$ ls
file1
sunnogo@a3e420:~/test/mv$ echo "second file" > file2
sunnogo@a3e420:~/test/mv$ echo "third file" > file3
sunnogo@a3e420:~/test/mv$ ls
file1  file2  file3

2. 重命名文件
没有覆盖的情况：
sunnogo@a3e420:~/test/mv$ mv file1 file4
sunnogo@a3e420:~/test/mv$ cat file4 
first file
sunnogo@a3e420:~/test/mv$ ls
file2  file3  file4

有覆盖的情况：
sunnogo@a3e420:~/test/mv$ mv file4 file1
sunnogo@a3e420:~/test/mv$ mv file1 file2
sunnogo@a3e420:~/test/mv$ ls
file2  file3
sunnogo@a3e420:~/test/mv$ cat file2
first file

有覆盖提示的情况：
sunnogo@a3e420:~/test/mv$ mv -i file2 file3
mv: overwrite ‘file3’? y
sunnogo@a3e420:~/test/mv$ 
sunnogo@a3e420:~/test/mv$ ls
file3
sunnogo@a3e420:~/test/mv$ cat file3 
first file
sunnogo@a3e420:~/test/mv$ 

3. 移动多个文件到同一个目录
sunnogo@a3e420:~/test/mv$ mv file3 file1
sunnogo@a3e420:~/test/mv$ echo "second file" > file2
sunnogo@a3e420:~/test/mv$ echo "third file" > file3
sunnogo@a3e420:~/test/mv$ 
sunnogo@a3e420:~/test/mv$ ls
file1  file2  file3
sunnogo@a3e420:~/test/mv$ mkdir dir
sunnogo@a3e420:~/test/mv$ mv -t dir/ file1 file2 file3 
sunnogo@a3e420:~/test/mv$ ls
dir
sunnogo@a3e420:~/test/mv$ ls dir/
file1  file2  file3
sunnogo@a3e420:~/test/mv$ 
```

> Written with [StackEdit](https://stackedit.io/).
