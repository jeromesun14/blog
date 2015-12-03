title: Linux命令行查看文件开头的内容 - head
date: 2015-01-04 22:48:18
toc: true
tags: [Linux命令, 字符流操作]
categories: shell
keywords: [linux, command, head]
description: Linux命令行文件开头的内容。
---

head - output the first part of files，用于输出文件开头的内容。

# 查看文件开头若干行的内容
使用`-n lines`选项显示文件开头的前lines行。

<!--more-->

```
sunnogo@a3e420:~/test/hello$ cat hello.c -n
     1  #include <stdio.h>
     2  /* just a comment */
     3  int main(void)
     4  {
     5      printf("Hello world!\n");
     6
     7      return 0;
     8  }
sunnogo@a3e420:~/test/hello$ head hello.c -n 5
#include <stdio.h>
/* just a comment */
int main(void)
{
    printf("Hello world!\n");
```

另一种head命令的使用方法： 

```
sunnogo@a3e420:~/test/hello$ cat hello.c -n | head -n 5
     1  #include <stdio.h>
     2  /* just a comment */
     3  int main(void)
     4  {
     5      printf("Hello world!\n");
sunnogo@a3e420:~/test/hello$ 
```

# 查看文件开头若干字节的内容
使用`-c bytes`显示文件前bytes字节的内容

```
sunnogo@a3e420:~/test/hello$ head -c 5 hello.c 
#incl
sunnogo@a3e420:~/test/hello$
```

# 输出文件开头内容的同时显示文件名
使用`-v`选项显示文件名：

```
sunnogo@a3e420:~/test/hello$ head -vn 5 hello.c 
==> hello.c <==
#include <stdio.h>
/* just a comment */
int main(void)
{
    printf("Hello world!\n");
```

