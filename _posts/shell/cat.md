title: Linux命令行浏览文件内容 - cat
date: 2014-12-31 23:48:18
toc: true
tags: [Linux命令, 文件系统]
categories: shell
keywords: [linux, command, cat]
description: Linux命令行浏览文件内容。
---

cat - concatenate files and print on the standard output，用来连接文件并显示出来。

# 基础用法

`cat file1 file2 ...`

如果文件很大，使用cat输出时会刷屏，甚至刷个不停。此时有多种处理方法：

* 如果只看前几行，使用[head][1]命令，`cat filename | head -n 行数`
* 如果只看后几行，使用[tail][2]命令，`cat filename | tail -n 行数`
* 如果要查看中间部分，使用[less][3]命令，`cat filename | less`，再使用[less][3]命令的快捷键查看内容。

<!--more-->

例1：查看hello.c文件的内容
```
sunnogo@a3e420:~/test/hello$ cat hello.c 
#include <stdio.h>
/* just a comment */
int main(void)
{
    printf("Hello world!\n");

    return 0;
}
```

例2：查看readme.txt文件的内容
```
sunnogo@a3e420:~/test/hello$ cat readme.txt 
This is a Hello world source file in C language.

Compile:
    gcc -o hello hello.c

    Run:
        ./hello
```

例3：同时查看hello.c和readme.txt的内容
```
sunnogo@a3e420:~/test/hello$ cat hello.c readme.txt 
#include <stdio.h>
/* just a comment */
int main(void)
{
    printf("Hello world!\n");

    return 0;
}
This is a Hello world source file in C language.

Compile:
        gcc -o hello hello.c

Run:
    ./hello

```

# 进阶使用
## 显示行号
`-n`选项，显示行号，`-b`选项可在显示行号时忽略空行。

例4：显示行号输出hello.c的内容
```
sunnogo@a3e420:~/test/hello$ cat -n hello.c 
     1  #include <stdio.h>
     2  /* just a comment */
     3  int main(void)
     4  {
     5      printf("Hello world!\n");
     6
     7      return 0;
     8  }
```

例5：显示行号输出hello.c的内容，计算行号时忽略空行。
```
sunnogo@a3e420:~/test/hello$ cat -nb hello.c 
     1  #include <stdio.h>
     2  /* just a comment */
     3  int main(void)
     4  {
     5      printf("Hello world!\n");

     6      return 0;
     7  }
sunnogo@a3e420:~/test/hello$ 
```

## 显示特殊字符

* `-T`显示TAB为`^I`，`-t`等于`-vT`
* `-E`在行尾显示`$`，`-e`等于`-vE`
* `-v`显示windows的换行`^M`
* `-A`等于`-vET`

例6：显示windows式的换行
```
sunnogo@a3e420:~/test/hello$ cat -v readme.txt 
This is a Hello world source file in C language.

Compile:
        gcc -o hello hello.c^M

Run:
    ./hello
```

例7：显示行尾，并同时显示windows式的换行
```
sunnogo@a3e420:~/test/hello$ cat -vE readme.txt 
This is a Hello world source file in C language.$
$
Compile:$
        gcc -o hello hello.c^M$
$
Run:$
    ./hello$
$
```

例8：显示TAB键、显示行尾并同时显示windows式的换行
```
sunnogo@a3e420:~/test/hello$ cat -vET readme.txt 
This is a Hello world source file in C language.$
$
Compile:$
^Igcc -o hello hello.c^M$
$
Run:$
    ./hello$
$
sunnogo@a3e420:~/test/hello$ 

```

[1]: http://sunnogo.tk/shell/head.html
[2]: http://sunnogo.tk/shell/tail.html
[3]: http://sunnogo.tk/shell/less.html

