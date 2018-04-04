title: Linux export 变量
date: 2018-04-04 14:53:00
toc: true
tags: [Linux]
categories: shell
keywords: [linux, export, /etc/profile, /etc/bash.bashrc, .bashrc]
description: Linux export 变量。
---

export 命令用于声明一个变量。变量的声明同样有工作域一说。例如：

* 如果在 terminal 中执行 `export ABC=abc`，则只在本终端中生效；
* 如果在 `~/.bashrc` 中写入 `export ABC=abc`，则在本用户生效；
* 如果在 `/etc/profile` 或 `/etc/bash.bashrc` 中写入 `export ABC=abc`，则全局生效。

注意 `/etc/rc.local` 中使用 export 命令，并不生效。具体原因不明。
