title: Linux命令行查看文件结尾的内容 - tail
date: 2015-01-04 22:48:18
toc: true
tags: [Linux命令, 字符流操作]
categories: shell
keywords: [linux, command, tail]
description: Linux命令行文件结尾的内容。
---

tail - output the last part of files，用于输出文件结尾的内容。  
tail命令的参数与head命令类似，支持显示结尾连续n行的内容或c字节的内容。

# 查看文件结尾若干行的内容
使用`-n lines`选项显示文件结尾的前lines行。

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
sunnogo@a3e420:~/test/hello$ tail hello.c -n 5
{
    printf("Hello world!\n");

    return 0;
}
sunnogo@a3e420:~/test/hello$
```

另一种使用方法： 

```
sunnogo@a3e420:~/test/hello$ cat hello.c -n | tail -n 5
     4  {
     5      printf("Hello world!\n");
     6
     7      return 0;
     8  }
sunnogo@a3e420:~/test/hello$ 
```

# 查看文件结尾若干字节的内容
使用`-c bytes`显示文件前bytes字节的内容

```
sunnogo@a3e420:~/test/hello$ tail -c 5 hello.c 
0;
}
```

# 输出文件结尾内容的同时显示文件名
使用`-v`选项显示文件名：

```
sunnogo@a3e420:~/test/hello$ tail -vn 5 hello.c 
==> hello.c <==
{
    printf("Hello world!\n");

    return 0;
}
sunnogo@a3e420:~/test/hello$ 
```

# 使用tail命令监控文件的实时状态

* `-f`选项，让tail命令一直保持活动状态，如果有新的内容加到文件的末尾就显示出来；
* `--pid=PID`选项，和`-f`参数一起使用，跟踪一个文件直到进程ID为PID的进程结束；
* `-s sec`选项，和`-f`参数一起使用，在每次循环输出之间休眠sec秒。

样例待后续补充。

