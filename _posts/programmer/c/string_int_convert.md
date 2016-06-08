title: C 语言字符串与整型相互转换
date: 2016-04-22 13:03:24
toc: true
tags: [Linux, c]
categories: c
keywords: [Linux, c, string, int, convert]
description: Linux c，字符串与整型数据相互转换。
---

## string to int

可参考 [stackoverflow 回答](http://stackoverflow.com/questions/10156409/convert-hex-string-char-to-int)。

* atoi / atol
* strtol，可用于多种进制转换
* sscanf

## int to string

* itoa
* sprintf / snprintf

<!--more-->

## MAC to string
本节来自 [stackoverflow](http://stackoverflow.com/questions/12772685/how-to-convert-mac-string-to-a-byte-address-in-c)：

On a C99-conformant implementation, this should work

```
unsigned char mac[6];

sscanf(macStr, "%hhx:%hhx:%hhx:%hhx:%hhx:%hhx", &mac[0], &mac[1], &mac[2], &mac[3], &mac[4], &mac[5]);
```

Otherwise, you'll need:

```
unsigned int iMac[6];
unsigned char mac[6];
int i;

sscanf(macStr, "%x:%x:%x:%x:%x:%x", &iMac[0], &iMac[1], &iMac[2], &iMac[3], &iMac[4], &iMac[5]);
for(i=0;i<6;i++)
    mac[i] = (unsigned char)iMac[i];
```
