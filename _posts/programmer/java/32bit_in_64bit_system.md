
title: 解决 Can't load IA 32-bit .dll on a AMD 64-bit platform
date: 2017-04-01 10:15:24
toc: true
tags: [java]
categories: java
keywords: [java, 32-bit, 64-bit, jre, UnsatisfiedLinkError, can't load, 无法加载, dll, 32, 64]
description: 解决64位系统上无法使用32位 dll 的问题
---

问题log：

```
C:\Users\R0\Downloads\win32>java -Xmx1024m -jar xxx.jar
java.lang.UnsatisfiedLinkError: C:\Users\R0\Downloads\win32\jxxx.dll: Can't load IA 32-bit .dll on a AMD 64-bit platform
```

通过 `-d32` 选项指定使用 32 位 JVM，同样出错：

```
C:\Users\R0\Downloads\win32>java -d32 -Xmx1024m -jar xxx.jar
Error: This Java instance does not support a 32-bit JVM.
Please install the desired version.
```

解决，下载 [32 位 JRE](http://javadl.oracle.com/webapps/download/AutoDL?BundleId=218831_e9e7ea248e2c4826b92b3f075a80e441)，并使用 32 位 java.exe 的绝对路径调用 java。

例如：

```
C:\Users\R0\Downloads\win32>"C:\Program Files (x86)\Java\jre1.8.0_121\bin\java.exe"  -Xmx1024m -jar xxx.jar
```
