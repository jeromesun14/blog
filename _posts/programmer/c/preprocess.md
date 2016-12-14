title: linux c 预处理使用记录
date: 2012-12-18 13:03:24
toc: true
tags: [linux, c]
categories: c
keywords: [linux, c, 预处理, proprocessing]
description: Linux c 预处理使用记录。
---

字符串化操作，Stringfication
---------------------------
符号 `#`，对它所引用的宏变量，通过替换后，在其左右各加上一个双引号。

```
#define WARN_IF(EXP)    do{ if (EXP)    fprintf(stderr, "Warning: " #EXP "/n"); }  while(0)
```

实际使用中会出现下面所示的替换过程：
```
WARN_IF (divider == 0);
```
被替换为：
```
do {
if (divider == 0)
fprintf(stderr, "Warning" "divider == 0" "/n");
} while(0);
```

连接符，Concatenator
-------------------

`##`，用来将两个token连接成一个Token，连接的Tocken不一定是宏变量。

