title: Linux 查看 CPU 信息
date: 2017-02-09 12:01:18
tags: [Linux, CPU]
categories: shell
keywords: [linux, cpu, cpuinfo, frequency, 频率, 核]
toc: true
description: Linux命令行查看 CPU 信息。
---

本文记录 Linux 操作系统中查看 CPU 信息的方法。查看的内容包含但不局限于：

* CPU 型号
* CPU 核数
* CPU 主频
* Cache 容量
* CPU 指令集
* CPU 特性

## `cat /proc/cpuinfo`

```
sunyongfeng@openswitch-OptiPlex-380:~$ cat /proc/cpuinfo 
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 23
model name      : Intel(R) Core(TM)2 Duo CPU     E7500  @ 2.93GHz
stepping        : 10
microcode       : 0xa07
cpu MHz         : 1600.000
cache size      : 3072 KB
physical id     : 0
siblings        : 2
core id         : 0
cpu cores       : 2
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 13
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx lm constant_tsc arch_perfmon pebs bts rep_good nopl aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 xsave lahf_lm tpr_shadow vnmi flexpriority dtherm
bugs            :
bogomips        : 5851.77
clflush size    : 64
cache_alignment : 64
address sizes   : 36 bits physical, 48 bits virtual
power management:

processor       : 1
vendor_id       : GenuineIntel
cpu family      : 6
model           : 23
model name      : Intel(R) Core(TM)2 Duo CPU     E7500  @ 2.93GHz
stepping        : 10
microcode       : 0xa07
cpu MHz         : 1600.000
cache size      : 3072 KB
physical id     : 0
siblings        : 2
core id         : 1
cpu cores       : 2
apicid          : 1
initial apicid  : 1
fpu             : yes
fpu_exception   : yes
cpuid level     : 13
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx lm constant_tsc arch_perfmon pebs bts rep_good nopl aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 xsave lahf_lm tpr_shadow vnmi flexpriority dtherm
bugs            :
bogomips        : 5851.77
clflush size    : 64
cache_alignment : 64
address sizes   : 36 bits physical, 48 bits virtual
power management:
```

## `dmesg | grep Hz`

```
sunyongfeng@openswitch-OptiPlex-380:~$ dmesg | grep Hz
[    0.000000] tsc: Detected 2925.887 MHz processor
[    0.104000] smpboot: CPU0: Intel(R) Core(TM)2 Duo CPU     E7500  @ 2.93GHz (family: 0x6, model: 0x17, stepping: 0xa)
[    0.280044] hpet0: 3 comparators, 64-bit 14.318180 MHz counter
[    1.840016] tsc: Refined TSC clocksource calibration: 2925.999 MHz
```

## `lscpu`

```
sunyongfeng@openswitch-OptiPlex-380:~$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    2
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 23
Model name:            Intel(R) Core(TM)2 Duo CPU     E7500  @ 2.93GHz
Stepping:              10
CPU MHz:               1600.000
CPU max MHz:           2933.0000
CPU min MHz:           1600.0000
BogoMIPS:              5851.77
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              3072K
NUMA node0 CPU(s):     0,1
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx lm constant_tsc arch_perfmon pebs bts rep_good nopl aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 xsave lahf_lm tpr_shadow vnmi flexpriority dtherm
sunyongfeng@openswitch-OptiPlex-380:~$ 
```

## lshw -c cpu

```
sunyongfeng@openswitch-OptiPlex-380:~$ sudo lshw -c cpu
[sudo] password for sunyongfeng: 
  *-cpu                   
       product: Intel(R) Core(TM)2 Duo CPU     E7500  @ 2.93GHz
       vendor: Intel Corp.
       physical id: 1
       bus info: cpu@0
       size: 1600MHz
       capacity: 2933MHz
       width: 64 bits
       capabilities: fpu fpu_exception wp vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx x86-64 constant_tsc arch_perfmon pebs bts rep_good nopl aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 xsave lahf_lm tpr_shadow vnmi flexpriority dtherm cpufreq
```

## `sudo dmidecode -t processor`

```
sunyongfeng@openswitch-OptiPlex-380:~$ sudo dmidecode -t processor
# dmidecode 3.0
Getting SMBIOS data from sysfs.
SMBIOS 2.5 present.

Handle 0x0400, DMI type 4, 40 bytes
Processor Information
        Socket Designation: CPU
        Type: Central Processor
        Family: Core 2 Duo
        Manufacturer: Intel
        ID: 7A 06 01 00 FF FB EB BF
        Signature: Type 0, Family 6, Model 23, Stepping 10
        Flags:
                FPU (Floating-point unit on-chip)
                VME (Virtual mode extension)
                DE (Debugging extension)
                PSE (Page size extension)
                TSC (Time stamp counter)
                MSR (Model specific registers)
                PAE (Physical address extension)
                MCE (Machine check exception)
                CX8 (CMPXCHG8 instruction supported)
                APIC (On-chip APIC hardware supported)
                SEP (Fast system call)
                MTRR (Memory type range registers)
                PGE (Page global enable)
                MCA (Machine check architecture)
                CMOV (Conditional move instruction supported)
                PAT (Page attribute table)
                PSE-36 (36-bit page size extension)
                CLFSH (CLFLUSH instruction supported)
                DS (Debug store)
                ACPI (ACPI supported)
                MMX (MMX technology supported)
                FXSR (FXSAVE and FXSTOR instructions supported)
                SSE (Streaming SIMD extensions)
                SSE2 (Streaming SIMD extensions 2)
                SS (Self-snoop)
                HTT (Multi-threading)
                TM (Thermal monitor supported)
                PBE (Pending break enabled)
        Version: Not Specified
        Voltage: 1.2 V
        External Clock: 1066 MHz
        Max Speed: 5200 MHz
        Current Speed: 2933 MHz
        Status: Populated, Enabled
        Upgrade: Socket LGA775
        L1 Cache Handle: 0x0700
        L2 Cache Handle: 0x0701
        L3 Cache Handle: Not Provided
        Serial Number: Not Specified
        Asset Tag: Not Specified
        Part Number: Not Specified
        Core Count: 2
        Core Enabled: 2
        Thread Count: 2
        Characteristics:
                64-bit capable
```

## `sudo watch -n 1  cat /sys/devices/system/cpu/cpu*/cpufreq/cpuinfo_cur_freq`

监控 CPU 频率变化。

```
sunyongfeng@openswitch-OptiPlex-380:~$ sudo watch -n 1  cat /sys/devices/system/cpu/cpu*/cpufreq/cpuinfo_cur_freq
Every 1.0s: cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_cu...  Thu Feb  9 12:50:54 2017

1600000
1600000
```
