title: octeon mips64 上使用 gprof 记录
date: 2017-06-08 23:34:00
toc: true
tags: [profiling]
categories: programmer
keywords: [linux, gcc, pg, gprof, multi thread, kill, SIGUSR1, mips64, gmon, empty, 无输出, 多线程, deamon, 性能, 调优]
description: octeon mips64 CPU 上使用 gprof 记录。
---

## 为何使用 gprof

对 mips64 设备上某业务进行性能调优，由于 Linux 系统为深度定制系统且 mips CPU 的限制，valgrind / 内核 perf 工具都有法使用（尚未研究 OProfile）。strace 只能 `trace system calls and signals`，因此考虑使用 gprof 进行性能调优。

## gprof 使用注意事项

* 编译和链接都需要使用`-pg`选项。若链接时没使用该选项，会输出空数据 gmon.out。

```
~ # ls -al gmon.out
-rw-r--r--    1 root     root           20 Jun  8 21:54 gmon.out
```

## 查看 octeon mips64 app 生成的 gmon.out
* 嵌入式设备上没有 gprof 程序
* 把 gmon.out 传到 x86 PC
* 通过交叉编译工具链中的 `mips64-octeon-linux-gnu-gprof` 查看

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test$mips64-octeon-linux-gnu-gprof ./rtxx -Q
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  ms/call  ms/call  name    
 23.64      0.13     0.13    79992     0.00     0.00  s_lookup
  3.64      0.15     0.02   553816     0.00     0.00  e_dump_prefix
  3.64      0.17     0.02   396896     0.00     0.00  db_get_key
  3.64      0.19     0.02    99999     0.00     0.00  e_dump_nsmmsg
  3.64      0.21     0.02    99999     0.00     0.01  e_handle
  3.64      0.23     0.02    79992     0.00     0.00  create_info
  3.64      0.25     0.02    79992     0.00     0.00  e_lkup_find_entry
  3.64      0.27     0.02    79991     0.00     0.00  e_proc
...
```

## 如何在不退出的业务中使用 gprof
详见 [Saving gmon.out before killing a process](https://stackoverflow.com/questions/10205543/saving-gmon-out-before-killing-a-process)。

gprof 通过 atexit 注册函数 `_mcleanup`，因此只要在退出前调用该函数即可。使用 SIGUSR1 信号，在信号处理函数中调用 `_mcleanup` 和 `_exit`。可正常运行！代码如下。

```c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>

void sigUsr1Handler(int sig)
{
    fprintf(stderr, "Exiting on SIGUSR1\n");
    void (*_mcleanup)(void);
    _mcleanup = (void (*)(void))dlsym(RTLD_DEFAULT, "_mcleanup");
    if (_mcleanup == NULL)
         fprintf(stderr, "Unable to find gprof exit hook\n");
    else _mcleanup();
    _exit(0);
}

int main(int argc, char* argv[])
{
    signal(SIGUSR1, sigUsr1Handler);
    neverReturningLibraryFunction();
}
```

### SIGUSR1 十进制值是多少？
不同的 arch 对 signal 的值定义不一样。在 x86 中的定义一般为 10，但是在 octeon mips 中就不一样了，详见 `arch/mips/include/asm/signal.h`：

```
#define SIGUSR1     16  /* User-defined signal 1 (POSIX).  */                                       
#define SIGUSR2     17  /* User-defined signal 2 (POSIX).  */                                       
```

所以，此时我只要 `kill -16 my_proc` 就会生成 gmon.out 了。

### 编译提示 Undefined reference to 'dlsym'

链接参数加上 `-ldl` 即可。据说 C++ 的链接参数是 `-Wl,--no-as-needed -ldl -pg`。

## 多线程使用 gprof

gprof 默认不支持多线程，通过 http://sam.zoy.org/writings/programming/gprof.html 提供的 gprof-helper 包装 pthread_create 函数，注入钩子后即可。

源代码如下。

* 通过 `gcc -shared -fPIC gprof-helper.c -o gprof-helper.so -lpthread -ldl` 编译该文件为共享库
* 通过 `LD_PRELOAD=./gprof-helper.so your_program` 应用该共享库

```c
/* gprof-helper.c -- preload library to profile pthread-enabled programs
 *
 * Authors: Sam Hocevar <sam at zoy dot org>
 *          Daniel J枚nsson <danieljo at fagotten dot org>
 *
 *  This program is free software; you can redistribute it and/or
 *  modify it under the terms of the Do What The Fuck You Want To
 *  Public License as published by Banlu Kemiyatorn. See
 *  http://sam.zoy.org/projects/COPYING.WTFPL for more details.
 *
 * Compilation example:
 * gcc -shared -fPIC gprof-helper.c -o gprof-helper.so -lpthread -ldl
 *
 * Usage example:
 * LD_PRELOAD=./gprof-helper.so your_program
 */

#define _GNU_SOURCE
#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>
#include <pthread.h>

static void * wrapper_routine(void *);

/* Original pthread function */
static int (*pthread_create_orig)(pthread_t *__restrict,
                                  __const pthread_attr_t *__restrict,
                                  void *(*)(void *),
                                  void *__restrict) = NULL;

/* Library initialization function */
void wooinit(void) __attribute__((constructor));

void wooinit(void)
{
    pthread_create_orig = dlsym(RTLD_NEXT, "pthread_create");
    fprintf(stderr, "pthreads: using profiling hooks for gprof\n");
    if(pthread_create_orig == NULL)
    {
        char *error = dlerror();
        if(error == NULL)
        {
            error = "pthread_create is NULL";
        }
        fprintf(stderr, "%s\n", error);
        exit(EXIT_FAILURE);
    }
}

/* Our data structure passed to the wrapper */
typedef struct wrapper_s
{
    void * (*start_routine)(void *);
    void * arg;

    pthread_mutex_t lock;
    pthread_cond_t  wait;

    struct itimerval itimer;

} wrapper_t;

/* The wrapper function in charge for setting the itimer value */
static void * wrapper_routine(void * data)
{
    /* Put user data in thread-local variables */
    void * (*start_routine)(void *) = ((wrapper_t*)data)->start_routine;
    void * arg = ((wrapper_t*)data)->arg;

    /* Set the profile timer value */
    setitimer(ITIMER_PROF, &((wrapper_t*)data)->itimer, NULL);

    /* Tell the calling thread that we don't need its data anymore */
    pthread_mutex_lock(&((wrapper_t*)data)->lock);
    pthread_cond_signal(&((wrapper_t*)data)->wait);
    pthread_mutex_unlock(&((wrapper_t*)data)->lock);

    /* Call the real function */
    return start_routine(arg);
}

/* Our wrapper function for the real pthread_create() */
int pthread_create(pthread_t *__restrict thread,
                   __const pthread_attr_t *__restrict attr,
                   void * (*start_routine)(void *),
                   void *__restrict arg)
{
    wrapper_t wrapper_data;
    int i_return;

    /* Initialize the wrapper structure */
    wrapper_data.start_routine = start_routine;
    wrapper_data.arg = arg;
    getitimer(ITIMER_PROF, &wrapper_data.itimer);
    pthread_cond_init(&wrapper_data.wait, NULL);
    pthread_mutex_init(&wrapper_data.lock, NULL);
    pthread_mutex_lock(&wrapper_data.lock);

    /* The real pthread_create call */
    i_return = pthread_create_orig(thread,
                                   attr,
                                   &wrapper_routine,
                                   &wrapper_data);

    /* If the thread was successfully spawned, wait for the data
     * to be released */
    if(i_return == 0)
    {
        pthread_cond_wait(&wrapper_data.wait, &wrapper_data.lock);
    }

    pthread_mutex_unlock(&wrapper_data.lock);
    pthread_mutex_destroy(&wrapper_data.lock);
    pthread_cond_destroy(&wrapper_data.wait);

    return i_return;
}
```

## 参考资料

1. [Saving gmon.out before killing a process](https://stackoverflow.com/questions/10205543/saving-gmon-out-before-killing-a-process)
2. [Undefined reference to 'dlsym'](https://stackoverflow.com/questions/20369672/undefined-reference-to-dlsym)
3. [Linux性能评测工具之一：gprof篇](http://blog.csdn.net/stanjiang2010/article/details/5655143)
