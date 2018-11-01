title: Linux shell 字符串处理
date: 2018-10-24 11:20:30
toc: true
tags: [Linux]
categories: shell
keywords: [linux, command, shell, string, 字符串]
description: Linux shell 脚本字符串处理笔记
---

* 首字母大写：`b=$(s^)`
* 截取几个字符：`${i:0:2}`

## 判断字符串子集

[Shell判断字符串包含关系的几种方法](https://blog.csdn.net/iamlihongwei/article/details/59484029):

* 通过 grep 判断
* 通过 `if [ A =~ B ]` 判断 A 包含 B。
