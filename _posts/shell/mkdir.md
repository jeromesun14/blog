title: Linux命令行创建目录 - mkdir
date: 2014-02-21 00:03:59
tags: [Linux命令, 文件系统]
categories: shell
keywords: [linux, command, mkdir, file system]
description: Linux命令行创建目录文件。
---

mkdir - make directories  
常用参数：  
`-p`，`--parents`，如果要创建的目录存在，也不会提示error，如果父目录不存在，则一并创建父目录。

<!--more-->

```
sunnogo@a3e420:~/test/mkdir$ ls 
sunnogo@a3e420:~/test/mkdir$ 
sunnogo@a3e420:~/test/mkdir$ mkdir first_dir
sunnogo@a3e420:~/test/mkdir$ ls -al
total 12
drwxr-xr-x  3 sunnogo sunnogo 4096  2月 23 22:09 .
drwxrwxr-x 12 sunnogo sunnogo 4096  2月 23 22:09 ..
drwxr-xr-x  2 sunnogo sunnogo 4096  2月 23 22:09 first_dir
sunnogo@a3e420:~/test/mkdir$ 
sunnogo@a3e420:~/test/mkdir$ mkdir second_dir/21dir
mkdir: cannot create directory ‘second_dir/21dir’: No such file or directory
<----创建多级目录如果不加-p选项且有父目录不存在，就会提示错误
sunnogo@a3e420:~/test/mkdir$ 
sunnogo@a3e420:~/test/mkdir$ mkdir -p second_dir/21dir
sunnogo@a3e420:~/test/mkdir$ ls -l
total 8
drwxr-xr-x 2 sunnogo sunnogo 4096  2月 23 22:09 first_dir
drwxr-xr-x 3 sunnogo sunnogo 4096  2月 23 22:10 second_dir
sunnogo@a3e420:~/test/mkdir$ ls -l *
first_dir:
total 0

second_dir:
total 4
drwxr-xr-x 2 sunnogo sunnogo 4096  2月 23 22:10 21dir
```

> Written with [StackEdit](https://stackedit.io/).
