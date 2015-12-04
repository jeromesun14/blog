title: Linux命令行创建文件、修改文件时间戳 - touch
date: 2014-02-21 00:03:59
tags: [Linux, 文件系统]
categories: shell
keywords: [linux, command, touch, file system]
description: Linux命令行创建文件、修改文件时间戳。
---

touch - change file timestamps  
touch常用于Linux命令行创建空文件，而从`man touch`的结果看来，touch主要用于修改时间戳，创建文件只是兼职工作。  

常用方式：

* `touch file`，如果当前目录内无该文件，则以当前时间创建空文件；如果当前目录内有该文件，则以当前时间修改文件时间戳（访问时间和修改时间）；
* `touch -t YearMonthDateHourMinute file`，指定时间修改文件时间戳，时间方式如201402232200。
* `touch -a file`，只修改访问时间（access timestamp）
* `touch -m file`，只修改修改时间（modify timestamp）

<!--more-->

```
sunnogo@a3e420:~/test/touch$ 
sunnogo@a3e420:~/test/touch$ touch first
sunnogo@a3e420:~/test/touch$ ls -l first 
-rw-r--r-- 1 sunnogo sunnogo 0  2月 23 21:56 first
sunnogo@a3e420:~/test/touch$ ls -lu first 
-rw-r--r-- 1 sunnogo sunnogo 0  2月 23 21:56 first
sunnogo@a3e420:~/test/touch$ 
sunnogo@a3e420:~/test/touch$ touch -at 201110101200 first 
sunnogo@a3e420:~/test/touch$ ls -l first 
-rw-r--r-- 1 sunnogo sunnogo 0  2月 23 21:56 first  <----修改时间戳不变
sunnogo@a3e420:~/test/touch$ ls -lu first 
-rw-r--r-- 1 sunnogo sunnogo 0 10月 10  2011 first  <----访问时间戳改变
sunnogo@a3e420:~/test/touch$ 
sunnogo@a3e420:~/test/touch$ touch -mt 201110101200 first 
sunnogo@a3e420:~/test/touch$ ls -l first 
-rw-r--r-- 1 sunnogo sunnogo 0 10月 10  2012 first  <----修改时间戳改变
sunnogo@a3e420:~/test/touch$ ls -lu first 
-rw-r--r-- 1 sunnogo sunnogo 0 10月 10  2011 first
```

> Written with [StackEdit](https://stackedit.io/).
