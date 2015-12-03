title: Linux命令行删除文件 - rm
date: 2014-02-21 00:03:59
tags: [Linux命令, 文件系统]
categories: shell
keywords: [linux, command, rm, file system]
description: Linux命令行删除普通文件、目录文件。
---

rm - remove files or directories  
命令语法：rm [option] file  
常见参数：
`-f`，`--force`，不提示直接删除。有风险，须谨慎。  
`-i`，删除每个文件前，先提示一把。如果批量删除多个文件的话，略显麻烦。  
`-I`，避免删除多个文件时，像-i那么麻烦。如果删除三个或以上文件，只会提醒一次。  
`-r`，删除目录以及递归删除目录下的文件和子目录。

本人较喜欢使用`rm -rf dir`的形式删除文件，也曾因此丢失重要数据，不过吃一堑长一智，也因此学会即时备份重要数据。

<!--more-->

```
sunnogo@a3e420:~/test/rm$ ls
sunnogo@a3e420:~/test/rm$ touch a
sunnogo@a3e420:~/test/rm$ rm a
<----普通删除不提示，直接删除，对比下面的-i删除
sunnogo@a3e420:~/test/rm$ ls
sunnogo@a3e420:~/test/rm$ touch b
sunnogo@a3e420:~/test/rm$ rm -i b
rm: remove regular empty file ‘b’? y

sunnogo@a3e420:~/test/rm$ ls
sunnogo@a3e420:~/test/rm$ mkdir -p dir1/dir1_1/dir1_1_1
sunnogo@a3e420:~/test/rm$ touch dir1/aa dir1/bb dir1/cc dir1/dd
sunnogo@a3e420:~/test/rm$ rm -ri dir1/
rm: descend into directory ‘dir1/’? y
rm: remove regular empty file ‘dir1/dd’? y
rm: remove regular empty file ‘dir1/bb’? y
rm: remove regular empty file ‘dir1/cc’? y
rm: descend into directory ‘dir1/dir1_1’? y
rm: remove directory ‘dir1/dir1_1/dir1_1_1’? y
rm: remove directory ‘dir1/dir1_1’? y
rm: remove regular empty file ‘dir1/aa’? y
rm: remove directory ‘dir1/’? y
<----批量删除的-i选项太麻烦了。

sunnogo@a3e420:~/test/rm$ 
sunnogo@a3e420:~/test/rm$ mkdir -p dir1/dir1_1/dir1_1_1
sunnogo@a3e420:~/test/rm$ touch dir1/aa dir1/bb dir1/cc dir1/dd
sunnogo@a3e420:~/test/rm$ rm -rI dir1/
rm: remove all arguments recursively? y
<----大写的-I选项轻松多了

sunnogo@a3e420:~/test/rm$ 
sunnogo@a3e420:~/test/rm$ mkdir -p dir1/dir1_1/dir1_1_1
sunnogo@a3e420:~/test/rm$ touch dir1/aa dir1/bb dir1/cc dir1/dd
sunnogo@a3e420:~/test/rm$ rm -rf dir1/
<----数据无价，强制删除需谨慎。
```

还有一个专门删除目录的命令`rmdir`，man手册中rmdir的说明是“rmdir - remove empty directories”，只能删除空目录，有rm的存在，rmdir看着有点鸡肋，不详述。

> Written with [StackEdit](https://stackedit.io/).
