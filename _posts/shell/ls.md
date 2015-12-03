title: Linux命令行浏览文件 - ls
date: 2014-02-21 00:03:59
tags: [Linux命令, 文件系统]
categories: shell
toc: true
keywords: [linux, command, ls, file system]
description: Linux命令行浏览文件属性信息。
---

ls - list directory content，用来查看文件属性信息。以下简要说明ls的常见使用方法。

# 常见文件属性信息汇总
ls命令主要用于查看文件的属性信息。Linux文件有以下几种主要属性。

1. 文件名
2. 文件类型，如普通文件、目录文件、软链接文件、字符设备文件、设设备文件、socket文件等。
3. 文件大小，单位可以是Byte、KByte、MByte、GByte或者block。
4. 是否隐藏文件，linux的隐藏文件的定义较为简单，文件名以“.”的文件都叫隐藏文件。ls命令默认不会输出隐藏文件的信息。
5. 文件权限，每三种用户类型：所属者（owner）、所属组（group）、其他人（other），每种用户有三种权限：可读、可写、可执行，可使用3 × 3 bits表示某种类型用户的某种权限。
6. owner信息，一般是名称或用户编号。
7. group信息，一般是名称或组编号。
8. 时间属性，含Access time、modify time、change time，分别以atime、mtime、ctime表示。
* access time，访问时间，读一次这个文件的内容，这个时间就会更新。比如对这个文件运用 more、cat等命令。ls、stat命令都不会修改文件的访问时间。
* modify time，修改时间是文件内容最后一次被修改时间。比如：vi后保存文件。ls -l列出的时间就是这个时间。
* change time，状态改动时间。是该文件的i节点最后一次被修改的时间，通过chmod、chown命令修改一次文件属性，这个时间就会更新。inode的概念将于《inode学习》一文详述。

<!--more-->

> 注意：Linux中什么都是文件，目录也只是一种文件。这个观点始终贯穿Linux操作系统。

# ls命令基础
## 查看文件名
直接敲一下ls命令就能看到当前目录下的所有非隐藏文件。
```
sunnogo@a3e420:~/test/hello$ ls
hello  hello.c  hello.i  hello.o  hello.s
```
## 查看隐藏文件
带`-a`参数，可查看列出隐藏文件。如以下的.hello.hide文件。

注意`.`和`..`，Linux中使用`.`表示当前目录，使用`..`表示上一级目录。看看`ls -a`、`ls -a .`和`ls -a ..`三者有什么差异。

```
sunnogo@a3e420:~/test/hello$ ls -a
.  ..  hello  hello.c  .hello.hide  hello.i  hello.o  hello.s
sunnogo@a3e420:~/test/hello$ ls -a .
.  ..  hello  hello.c  .hello.hide  hello.i  hello.o  hello.s
sunnogo@a3e420:~/test/hello$ ls -a ..
.  ..  cmd_line  .config  config_file  .git  .gitignore  hello  .hello.c.swp  mkdir  mv  rm  script  tdef  touch

```

## 查看文件大小
ls命令使用`-s`参数查看文件大小，默认以block为单位。使用`-h`参数将文件大小的单位转换为human readable的单位，即常见的KB、MB、GB等。

ls的`-l`参数，即long list formating，把大多数的文件属性信息都显示出来，因此一般只记`-l`参数，而无需去记类似`-s`这样的参数。注意`-l`参数默认不显示隐藏文件，默认的文件大小以Byte为单位。
```
sunnogo@a3e420:~/test/hello$ ls -s
total 40
 8 hello   4 hello.c  20 hello.i   4 hello.o   4 hello.s
sunnogo@a3e420:~/test/hello$ ls -sh
total 40K
8.0K hello  4.0K hello.c   20K hello.i  4.0K hello.o  4.0K hello.s
sunnogo@a3e420:~/test/hello$ ls -alh
total 48K
drwxrwxr-x  2 sunnogo sunnogo 4.0K Dec 20 22:00 .
drwxrwxr-x 14 sunnogo sunnogo 4.0K Apr 20  2014 ..
-rwxrwxr-x  1 sunnogo sunnogo 7.2K Dec 20 22:00 hello
-rw-rw-r--  1 sunnogo sunnogo  104 Jul 24  2013 hello.c
-rw-rw-r--  1 sunnogo sunnogo    0 Dec 20 22:00 .hello.hide
-rw-rw-r--  1 sunnogo sunnogo  18K Jul 26  2013 hello.i
-rw-rw-r--  1 sunnogo sunnogo 1.1K Jul 26  2013 hello.o
-rw-rw-r--  1 sunnogo sunnogo  491 Jul 26  2013 hello.s
```

## 查看文件权限信息
如上所示，`-l`参数的第一列显示文件权限信息，第一位表示文件类型，后九位表示文件权限。
以文件hello为例，其权限信息`rwxrwxr-x`表示：

* 第1到3位`rwx`，对象是文件所属者，所属者对该文件有可读、可写、可执行三种权限；
* 第4到6位`rwx`，对象是文件所属组，组是多用户组成的，所属组对该文件有可读、可写、可执行三种权限。
* 第7到9位`r-x`，对象是其他用户，其他用户对该文件有可读、可执行两种权限，没有权限运行该文件。

对文件的属性Linux系统上还有另一种用数值表示的方法，即通过3个bit来表示一种用户的权限，r是最高位，w是第二位，x是最低位，对应位置为1表示用户拥有对应的权限，0表示无此权限。例如，数字6（即110）表示可读、可写两种权限，数字5（即101）表示可读、可执行两种权限。这种表示方法在`chmod`命令中经常使用。

## 查看文件类型
`-l`参数即可查看文件类型，第一位的第一位表示文件类型，含目录（d），文件（-），字符型文件（c），块文件（b），软链接文件（l）等。  

## 查看文件时间属性
可用`stat`命令查看文的三种时间属性。ls命令也可查看时间属性，默认输出mtime，-c参数表示ctime，-u参数表示atime。

* 查看ctime，`ls -lc filename`
* 查看atime，`ls -lu filename`
* 查看mtime，`ls -l filename`

## 排序相关参数
`-t`，以文件的时间排序，默认是最后修改时间，如果有-u选项，则是以上次访问时间排序。  
`-S`，大写S，以文件大小排序输出，注意，目录文件的大小始终是4KB，不会去统计目录内部文件的总大小。  
`-u`，小写u，输出文件的最后访问时间，而非最后修改时间。  
`-X`，大写X，按文件扩展名排序输出。  

## 其他参数
* `-R`，即`--recursive`，递归列出子目录内容。这个参数在其他命令中也很常见，表示递归处理子目录。

* `-B`，即`--ignore-backups`，不显示以波浪线结尾的条目，波浪线是用来表示备份的副本，如果有用过gedit或kate，对这波浪线肯定深有体会。  
* `-i`，即`--inode`，显示每个文件的索引值。这个在学操作系统时可能能用上。  
* `-n`，即`--numeric-uid-gid`，显示数字类型的userid和groupid来替代名字。  

>TIPs：
>
> * *命令行参数*一般有单字母参数和全字参数两种形式，比如`ls -a`和`ls --all`。单字母通常由英文破折号开始，全字参数则更容易看懂，通常以双英文破折号开始。许多参数都有单字母和全字两种版本，而有些则只有一种。
> * ls命令支持*标准通配符*，问号代表一个字符，星号代表零或多个字符。
> * 如果不知道某个命令怎么使用，要学会使用man手册，即Linux的帮助手册。通过命令`man your_command`即可查看对应命令的使用方法、参数说明等。

</br>
> Written with [StackEdit](https://stackedit.io/).


