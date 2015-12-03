title: Linux命令行查找文件 - find
date: 2014-12-25 23:01:18
tags: [Linux命令, 文件系统]
categories: shell
keywords: [linux, command, find]
toc: true
description: Linux命令行查找文件
---

find - search for files in a directory hierarchy  
按照目录等级搜索文件。

基本命令格式：

```
find [path...] [expression]
```

# 基础用法
## 查找本目录下特定名称
`find . -name filename`可用于查找本目录下一个特定名称的文件。`.`表示当前路径，`filename`表示待查找的文件。下文几乎所有的例子的path都使用当前路径。

```
查找特定文件hello.c
sunnogo@a3e420:~/test$ find . -name hello.c
./hello/hello.c
```
<!--more-->

## 查找特定后缀的文件
上述`filename`支持通配符，因此可简单实现按类型查找文件等功能。
```
查找后缀为.c的文件
sunnogo@a3e420:~/test$ find . -name *.c
./script/prof.c
./hello/hello.c
./tdef/test.c
./cmd_line/config_file/parse_conf.c
./config_file/parse_conf.c
```
## 查找模糊文件名的文件
在使用过程中，大家可能不记得某个文件的全名，但是大概记得文件名的某一段关键字符。这时通配符就又被派上用场了：
```
查找名字带parse的文件
sunnogo@a3e420:~/test$ find . -name *parse*
./cmd_line/config_file/parse_conf
./cmd_line/config_file/parse_conf.c
./config_file/parse_conf
./config_file/parse_conf.c
```

# 进阶用法

如果find命令的参数后接的数值，则有如下几种表示方式：

* `+n`，表示大于n
* `-n`，表示小于n
* `n`，表示等于n

## 按大小查找文件

`find . [-name filename] -size [+|-]n[b|c|w|k|M|G]`
其中：

* `-size`，表示只输出文件大小大于/小于/等于特定值n的文件
* 大小的单位如下，注意此处单位的大小写，k是小写的。
    * b表示512字节的块
    * c表示字节
    * w表示双字节
    * k表示1024字节
    * M表示1024kB
    * G表示1024MB

例子中用到的-ls参数是非posix标准参数，这里只是为了更好展示查找结果才使用。

```
查找当前目录及子目录中，所有文件大小大于4KB的文件
sunnogo@a3e420:~/test$ find . -size +4k -ls
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello/hello
8130650   20 -rw-rw-r--   1 sunnogo  sunnogo     17967 Jul 26  2013 ./hello/hello.i
7219878   12 -rw-r--r--   1 sunnogo  sunnogo     12288 Jul 24  2013 ./.hello.c.swp
7209916    8 -rwxrwxr-x   1 sunnogo  sunnogo      7122 Jan 17  2013 ./tdef/test
7340928    8 -rwxrwxr-x   1 sunnogo  sunnogo      7702 Mar 19  2013 ./cmd_line/config_file/parse_conf
7996916    8 -rwxrwxr-x   1 sunnogo  sunnogo      7702 Mar 19  2013 ./config_file/parse_conf

查找当前目录及子目录中，所有文件大小小于4KB的文件
sunnogo@a3e420:~/test$ find . -size -4k -ls
7340612    4 -rwxrwxr-x   3 sunnogo  sunnogo        94 Jun 21  2013 ./script/link.sh
7340612    4 -rwxrwxr-x   3 sunnogo  sunnogo        94 Jun 21  2013 ./script/if.sh
7340612    4 -rwxrwxr-x   3 sunnogo  sunnogo        94 Jun 21  2013 ./script/lnlink.sh
7340881    0 lrwxrwxrwx   1 sunnogo  sunnogo         5 Feb 23  2014 ./script/symbol.sh -> if.sh
7340895    0 lrwxrwxrwx   1 sunnogo  sunnogo         5 Feb 23  2014 ./script/lnsymbol.sh -> if.sh
7340899    4 -rw-rw-r--   1 sunnogo  sunnogo       519 Nov 12  2013 ./script/prof.c
...

查找当前目录及子目录文件中，所有文件大小等于4KB的文件。
sunnogo@a3e420:~/test$ find . -size 4k -ls
7209663    4 drwxrwxr-x  11 sunnogo  sunnogo      4096 Dec 26 00:22 .
7340537    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Feb 23  2014 ./script
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 ./hello
7342618    4 drwxr-xr-x   3 sunnogo  sunnogo      4096 Feb 23  2014 ./mv
7342622    4 drwxr-xr-x   2 sunnogo  sunnogo      4096 Feb 23  2014 ./mv/dir
7341100    4 drwxr-xr-x   2 sunnogo  sunnogo      4096 Feb 23  2014 ./touch
...
```

## 按权限查找文件

Linux文件权限的格式有两种，这在[ls][1]一文的“查看文件权限信息”中有详细说明。每个文件供给三种用户使用（owner、group、other），每种用户有可读（readable）、可写（writable）、可执行（executable）三种权限。因此一个文件有九种基本权限。

* 数值格式。假设有个文件的权限是“751”，则“7”表示文件的owner可读、可写、可执行，“5”表示文件所属group可读、可执行，“1”表示other用户可执行。
* 符号格式，以r表示可读、w表示可写、x表示可执行。一般配以o表示owner、g表示group、o表示other。

find命令按文件权限查找文件有两种格式：

* `find -perm -mode`，All of the permission bits mode are set for the file，即目标文件的权限要匹配到mode中所有允许的权限，但未涉及的权限可有可无；
* `find -perm /mode`，Any of the permission bits mode are set for the file，即查找到的文件只要匹配到mode中允许的一种权限即可；

如下例中的`find . -perm -700`表示查找权限至少是owner可读、可写、可执行的文件。如`ls`显示的结果，只有文件hello满足此条件。注意文件hello还包含group可读、可写可执行权限以及other的可读、可执行权限。
`find . -perm /700`的结果与`find . -perm -700`的结果就有明显差异，几乎把本目录中的所有文件都打出来了。

```
sunnogo@a3e420:~/test/hello$ ls -al
total 48
drwxrwxr-x  2 sunnogo sunnogo  4096 Dec 20 22:00 .
drwxrwxr-x 11 sunnogo sunnogo  4096 Dec 26 00:22 ..
-rwxrwxr-x  1 sunnogo sunnogo  7292 Dec 20 22:00 hello
-rw-rw-r--  1 sunnogo sunnogo   104 Jul 24  2013 hello.c
-rw-rw-r--  1 sunnogo sunnogo     0 Dec 20 22:00 .hello.hide
-rw-rw-r--  1 sunnogo sunnogo 17967 Jul 26  2013 hello.i
-rw-rw-rw-  1 sunnogo sunnogo  1028 Jul 26  2013 hello.o
-rw-rw-r--  1 sunnogo sunnogo   491 Jul 26  2013 hello.s

sunnogo@a3e420:~/test/hello$ find . -perm -700 -ls 
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 .
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello

sunnogo@a3e420:~/test/hello$ find . -perm /700 -ls 
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 .
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello
8127134    0 -rw-rw-r--   1 sunnogo  sunnogo         0 Dec 20 22:00 ./.hello.hide
8131706    4 -rw-rw-rw-   1 sunnogo  sunnogo      1028 Jul 26  2013 ./hello.o
8130651    4 -rw-rw-r--   1 sunnogo  sunnogo       491 Jul 26  2013 ./hello.s
7219321    4 -rw-rw-r--   1 sunnogo  sunnogo       104 Jul 24  2013 ./hello.c
8130650   20 -rw-rw-r--   1 sunnogo  sunnogo     17967 Jul 26  2013 ./hello.i

sunnogo@a3e420:~/test/hello$ find . -perm /003 -ls
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 .
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello
8131706    4 -rw-rw-rw-   1 sunnogo  sunnogo      1028 Jul 26  2013 ./hello.o
```

mode也可以使用符号格式，如下例子使用符号格式实现`find . -perm -700与find . -perm /003`。

```
等同“find . -perm -700”
sunnogo@a3e420:~/test/hello$ find . -perm -u+r,u+w,u+x -ls
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 .
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello

等同“find . -perm /003”
sunnogo@a3e420:~/test/hello$ find . -perm /o+w,o+x -ls
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 .
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello
8131706    4 -rw-rw-rw-   1 sunnogo  sunnogo      1028 Jul 26  2013 ./hello.o
```

## 按时间查找文件
Linux文件相关时间属性详见[ls][1]一文中“常见文件属性信息汇总”小节的描述。

* 按访问时间查找（access time，atime）
  * `-amin n`，查找在此前第[n - 1, n]分钟的期间被访问的文件（当n不等于0时为如上结论，如果n等于0，默认无输出，该文件的access time是否在未来的时间）。
  * `-anewer file`，查找access time比文件file晚的文件
  * `-atime n`，查找在此前第[(n - 1)× 24, n × 24]小时的期间被访问的文件

注意本章一开头对find的数值参数n的描述，+n表示大于n的情况，-n表示小于n的情况，同样适合于本小节所描述的参数。用于实现**查找某个时间点被访问以前或之后的文件**，这才是我们最想要的用途。注意此时的时间不再是个区间，而就是个时间点。具体用法如下例所述。

上述的时间期间可能不好理解，详见本文后面的小节“理解按时间查找文件中时间期间的含义”。

```
sunnogo@a3e420:~/test/hello$ touch -at 12262300 hello.i
sunnogo@a3e420:~/test/hello$ stat hello.i | grep "Access: 2014"
Access: 2014-12-26 23:00:00.000000000 +0800

sunnogo@a3e420:~/test/hello$ date
Fri Dec 26 23:14:08 CST 2014

sunnogo@a3e420:~/test/hello$ find . -amin 15
./hello.i
sunnogo@a3e420:~/test/hello$ find . -amin -14
sunnogo@a3e420:~/test/hello$ find . -amin -15
./hello.i
```


* 按状态变动时间查找（change time，ctime），命令与access time类似
  * `-cmin n`
  * `-cnewer file`
  * `-ctime n`
* 按修改时间查找（modify time，mtime）
  * `-mmin n`
  * `-mnewer file`，`-newer file`看似功能与本选项一致。
  * `-mtime n`

## 按文件类型查找文件
Linux文件相关时间属性详见[ls][1]一文中“常见文件属性信息汇总”小节的描述。

`find PATH -type file_type`，文件类型file_type包含如下几种：

* b，块设备文件
* c，字符设备文件
* d，目录文件
* p，有名管道文件
* f，普通文件
* l，符号链接文件
* s，socket文件

以下是样例：
```
sunnogo@a3e420:~/test$ find . -type d -ls
7209663    4 drwxrwxr-x  11 sunnogo  sunnogo      4096 Dec 26 00:22 .
7340537    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Feb 23  2014 ./script
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 ./hello
...

sunnogo@a3e420:~/test$ find . -type l -ls
7340881    0 lrwxrwxrwx   1 sunnogo  sunnogo         5 Feb 23  2014 ./script/symbol.sh -> if.sh
7340895    0 lrwxrwxrwx   1 sunnogo  sunnogo         5 Feb 23  2014 ./script/lnsymbol.sh -> if.sh
```

## 多条件查找

`find`命令支持运算符，允许多个expression进行计算。主要运算符有：

* `()`，和四则运算一样，表示优先执行括号内的表达式；
* `! expr`，类似逻辑运算的not，表示对表达式expr的结果取反；
* `expr1 -a expr2`，类似逻辑运算的and，表示expr1和expr2进行与运算。类似C语言的短路求值(short-circuit evalution)，若expr1为假，就不会计算expr2；
* `expr1 -o expr2`，类似逻辑运算的or，表示expr1和expr2进行或运行。同样使用短路求值，若expr1为真，则不会计算expr2

事实上`find`命令也有`-not`、`-and`、`-or`选项等同于上述三种运算符，但是这三种运算符是非POSIX兼容的，使用时需要注意。

```
sunnogo@a3e420:~/test$ ls -al hello/
total 48
drwxrwxr-x  2 sunnogo sunnogo  4096 Dec 20 22:00 .
drwxrwxr-x 12 sunnogo sunnogo  4096 Dec 26 23:27 ..
-rwxrwxr-x  1 sunnogo sunnogo  7292 Dec 20 22:00 hello
-rw-rw-r--  1 sunnogo sunnogo   104 Jul 24  2013 hello.c
-rw-rw-r--  1 sunnogo sunnogo     0 Dec 20 22:00 .hello.hide
-rw-rw-r--  1 sunnogo sunnogo 17967 Dec 26 23:02 hello.i
-rw-rw-rw-  1 sunnogo sunnogo  1028 Jul 26  2013 hello.o
-rw-rw-r--  1 sunnogo sunnogo   491 Jul 26  2013 hello.s

查找文件名为"hello"开头、且至少有一种用户有写权限的文件
sunnogo@a3e420:~/test$ find . -name "hello*" -a -perm /a+w -ls
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 ./hello
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello/hello
8131706    4 -rw-rw-rw-   1 sunnogo  sunnogo      1028 Jul 26  2013 ./hello/hello.o
8130651    4 -rw-rw-r--   1 sunnogo  sunnogo       491 Jul 26  2013 ./hello/hello.s
7219321    4 -rw-rw-r--   1 sunnogo  sunnogo       104 Jul 24  2013 ./hello/hello.c
8130650   20 -rw-rw-r--   1 sunnogo  sunnogo     17967 Dec 26 23:02 ./hello/hello.i

查找文件名为"hello"开头、且所有用户都有写权限的文件
sunnogo@a3e420:~/test$ find . -name "hello*" -a -perm -a+w -ls
8131706    4 -rw-rw-rw-   1 sunnogo  sunnogo      1028 Jul 26  2013 ./hello/hello.o

查找文件名为"hello"开头、且不是所有用户都有写权限的文件
sunnogo@a3e420:~/test$ find . -name "hello*" -a ! -perm -a+w -ls
8130648    4 drwxrwxr-x   2 sunnogo  sunnogo      4096 Dec 20 22:00 ./hello
8127133    8 -rwxrwxr-x   1 sunnogo  sunnogo      7292 Dec 20 22:00 ./hello/hello
8130651    4 -rw-rw-r--   1 sunnogo  sunnogo       491 Jul 26  2013 ./hello/hello.s
7219321    4 -rw-rw-r--   1 sunnogo  sunnogo       104 Jul 24  2013 ./hello/hello.c
8130650   20 -rw-rw-r--   1 sunnogo  sunnogo     17967 Dec 26 23:02 ./hello/hello.i

```

## 对查找结果进行处理
即对查找的结果作为参数输入给其他命令。  
有以下四种形式的格式，`\`只是个转义字符，为防止符号`;`或`+`在shell中有不同的含义。

* `-exec command "{}" \;`
* `-exec command "{}" \+`
* `-execdir command "{}" \;`
* `-execdir command "{}" \+`

目前尚不需理解命令参数结束符`;`与`+`的差异以及`exec`与`execdir`的差异。

```
sunnogo@a3e420:~/test$ mkdir exec
sunnogo@a3e420:~/test$ cd exec
sunnogo@a3e420:~/test/exec$ echo "abcdefg" > abc.txt
sunnogo@a3e420:~/test/exec$ echo "xyzabc" > xyz.txt
sunnogo@a3e420:~/test/exec$ echo "bcdefgh" > bcd.txt
sunnogo@a3e420:~/test/exec$ mkdir exec_sub
sunnogo@a3e420:~/test/exec$ cd exec_abc/
sunnogo@a3e420:~/test/exec/exec_abc$ echo "abcdefghijkl" > bck.txt
sunnogo@a3e420:~/test/exec/exec_abc$ cd ..

sunnogo@a3e420:~/test/exec$ find . -name "*bc*" 
./bcd.txt
./exec_abc
./exec_abc/bck.txt
./abc.txt

sunnogo@a3e420:~/test/exec$ find . -name "*bc*" -exec cat "{}" \+   
bcdefgh
cat: ./exec_abc: Is a directory
abcdefghijkl
abcdefg
```

## 按格式输出查找结果
使用`print`、`fprint`参数实现，相对的内容较多但是不重要，在此不详细，需要的同学使用`man find`看一下。

## 使用正则表达式匹配文件名
`-regex pattern`文件名可以用正则表达式匹配，这里不详述。

## 其他参数
还可以根据硬连接数、inode信息、owner信息等查找文件

## 理解按时间查找文件中时间期间的含义

“查找在此前第[n-1, n]分钟的期间被访问的文件”，不好理解，先看下面的例子：

首先，查看文件hello.i的属性信息，发现access time是22:32:52

```
sunnogo@a3e420:~/test/hello$ stat hello.i 
  File: ‘hello.i’
  Size: 17967           Blocks: 40         IO Block: 4096   regular file
Device: 805h/2053d      Inode: 8130650     Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/ sunnogo)   Gid: ( 1000/ sunnogo)
Access: 2014-12-26 22:32:52.248847993 +0800
Modify: 2014-12-26 22:32:50.072847903 +0800
Change: 2014-12-26 22:32:50.072847903 +0800
 Birth: -
```

其次，查看当前时间点，现在是22:57:38，距离hello.i的atime为24分钟46秒。

```
sunnogo@a3e420:~/test/hello$ date
Fri Dec 26 22:57:38 CST 2014
```

第三，看n值为24、25、26时`find . -amin n`的结果。

```
sunnogo@a3e420:~/test/hello$ find . -amin 25
./hello.i
sunnogo@a3e420:~/test/hello$ find . -amin 24
sunnogo@a3e420:~/test/hello$ find . -amin 26
```

接下来看看`find . -amin 0`会不会显示未来时间。实际上最科学的方法是去查找find命令的代码实现，这里仅仅是玩一玩，验证自己觉得可能比较有趣的情况，无需较真。

```
sunnogo@a3e420:~/test/hello$ date
Fri Dec 26 23:06:55 CST 2014

sunnogo@a3e420:~/test/hello$ touch -at 12300000 hello.i 
sunnogo@a3e420:~/test/hello$ stat hello.i | grep "Access: 2014"
Access: 2014-12-30 00:00:00.000000000 +0800

sunnogo@a3e420:~/test/hello$ find . -amin 0
sunnogo@a3e420:~/test/hello$ 
```

[1]: http://sunnogo.tk/shell/ls.html

> Written with [StackEdit](https://stackedit.io/).

