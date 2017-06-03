title: Linux 查看进程信息
date: 2017-06-03 13:05:18
tags: [Linux, 进程]
categories: shell
keywords: [linux, cpu, core, 核, 进程, process, rss, 内存, 使用率, memory, ps, top, proc, thread, 线程, 核心, 优先级, priority]
toc: true
description: Linux命令行查看进程信息。
---

## 查看 CPU 使用率

## 查看当前正在使用哪个 CPU core

top 命令中，`f` 命令添加域，选择 `j` 添加一列 P，显示上次使用的 SMP。`J: P          = Last used cpu (SMP)`。域前面有 "*" 表示已选中。

* 查看前

```
~ # top
top - 13:12:34 up  1:23,  0 users,  load average: 11.37, 11.21, 11.10
Tasks: 105 total,   3 running, 102 sleeping,   0 stopped,   0 zombie
Cpu(s): 23.3%us, 76.0%sy,  0.0%ni,  0.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    995508k total,   508256k used,   487252k free,        0k buffers
Swap:        0k total,        0k used,        0k free,    61552k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND            
 1546 root      20   0  456m 9592 3536 S  188  1.0 160:05.60 d        
  888 root      20   0     0    0    0 R  100  0.0  83:36.19 m    
 1781 root      20   0     0    0    0 R  100  0.0  83:16.54 c    
 1537 root      20   0 1481m 140m  37m S    8 14.4   4:57.15 a        
 1979 root      20   0  2848 1164  892 R    2  0.1   0:00.04 top                
    1 root      20   0  2004  696  592 S    0  0.1   0:02.49 init               
    2 root      20   0     0    0    0 S    0  0.0   0:00.00 kthreadd 
```

* `f`

```
Current Fields:  AEHIOQTWKNMbcdfgJplrsuvyzX  for window 1:Def
Toggle fields via field letter, type any other key to return 

* A: PID        = Process Id              u: nFLT       = Page Fault count
* E: USER       = User Name               v: nDRT       = Dirty Pages count
* H: PR         = Priority                y: WCHAN      = Sleeping in Function
* I: NI         = Nice value              z: Flags      = Task Flags <sched.h>
* O: VIRT       = Virtual Image (kb)    * X: COMMAND    = Command name/line
* Q: RES        = Resident size (kb)
* T: SHR        = Shared Mem size (kb)  Flags field:
* W: S          = Process Status          0x00000001  PF_ALIGNWARN
* K: %CPU       = CPU usage               0x00000002  PF_STARTING
* N: %MEM       = Memory usage (RES)      0x00000004  PF_EXITING
* M: TIME+      = CPU Time, hundredths    0x00000040  PF_FORKNOEXEC
  b: PPID       = Parent Process Pid      0x00000100  PF_SUPERPRIV
  c: RUSER      = Real user name          0x00000200  PF_DUMPCORE
  d: UID        = User Id                 0x00000400  PF_SIGNALED
  f: GROUP      = Group Name              0x00000800  PF_MEMALLOC
  g: TTY        = Controlling Tty         0x00002000  PF_FREE_PAGES (2.5)
* J: P          = Last used cpu (SMP)     0x00008000  debug flag (2.5)
  p: SWAP       = Swapped size (kb)       0x00024000  special threads (2.5)
  l: TIME       = CPU Time                0x001D0000  special states (2.5)
  r: CODE       = Code size (kb)          0x00100000  PF_USEDFPU (thru 2.4)
  s: DATA       = Data+Stack size (kb)
```

* 查看

```
top - 13:12:30 up  1:23,  0 users,  load average: 11.40, 11.21, 11.10
Tasks: 105 total,   4 running, 101 sleeping,   0 stopped,   0 zombie
Cpu(s): 25.6%us, 74.4%sy,  0.0%ni,  0.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    995508k total,   508256k used,   487252k free,        0k buffers
Swap:        0k total,        0k used,        0k free,    61552k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  P COMMAND          
 1546 root      20   0  456m 9592 3536 S  174  1.0 159:57.14 0 d      
  888 root      20   0     0    0    0 R  100  0.0  83:31.86 2 m  
 1781 root      20   0     0    0    0 R  100  0.0  83:12.20 3 c  
 1537 root      20   0 1481m 140m  37m S   16 14.4   4:56.98 1 a      
 1978 root      20   0  2912 1260  984 R   11  0.1   0:00.08 1 top              
    1 root      20   0  2004  696  592 S    0  0.1   0:02.49 0 init             
    2 root      20   0     0    0    0 S    0  0.0   0:00.00 1 kthreadd   
```

## 跟踪系统调用和信号 - strace

* strace -p xxx -c，ctrl + c 退出后列出运行期间的所有调用情况统计
* strace -p xxx，实时显示系统调用信息
