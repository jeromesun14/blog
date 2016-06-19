title: C程序异常提示“stack smashing detected”
date: 2016-06-19 15:38:24
toc: true
tags: [Linux, c]
categories: c
keywords: [Linux, c, overflow, 溢出, SIGABRT, smashing]
description: Linux C程序异常提示“stack smashing detected”。
---

如题，在程序运行中出现：

```
*** stack smashing detected ***: /home/sunyongfeng/workshop/test/ss_redis.x86/ssc_ef/ef.elf terminated

Program received signal SIGABRT, Aborted.
```

原因： memcpy 拷贝溢出，拷贝超过预期的大小。

详见 [stackoverflow](http://stackoverflow.com/questions/1345670/stack-smashing-detected) 的 sud03r 回答。以下引用该回答。
> Stack Smashing here is actually caused due to a protection mechanism used by gcc to detect buffer overflow errors. For example in the following snippet:
>
```
#include <stdio.h>

void func()
{
    char array[10];
    gets(array);
}

int main(int argc, char **argv)
{
    func();
}
```
>
> The compiler, (in this case gcc) adds protection variables (called canaries) which have known values. An input string of size greater than 10 causes corruption of this variable resulting in SIGABRT to terminate the program.
> 
> To get some insight, you can try disabling this protection of gcc using option   -fno-stack-protector  while compiling. In that case you will get a different error, most likely a segmentation fault as you are trying to access an illegal memory location. Note that -fstack-protector should always be turned on for release builds as it is a security feature.
> 
> You can get some information about the point of overflow by running the program with a debugger. Valgrind doesn't work well with stack-related errors, but like a debugger, it may help you pin-point the location and reason for the crash.
