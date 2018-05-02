title: Linux shell 脚本
date: 2018-04-28 16:07:00
toc: true
tags: [Linux]
categories: shell
keywords: [linux, command, shell, script, 脚本, if, 顺序, 循环, while, for, test]
description: Linux shell 脚本使用记录
---


## 函数

```bash
# Function: wait until ifindex ok
wait_ifindex() {
    while true; do
        if [ -e /sys/class/net/$1/ifindex ]; then
            IFINDEX=$(cat /sys/class/net/$1/ifindex)
            echo "Get interface ${1} ifindex ${IFINDEX}"
            if [ "$IFINDEX"x != "0"x ]; then
                break;
            fi
        fi
        echo "Wait for interface ${1} ifindex OK for 1 second..."
        sleep 1
    done
}

wait_ifindex eth0
```

输出：

```
~$ ./test.sh 
Get interface eth0 ifindex 2
```

## 返回值

`$#` 返回上一个命令的执行结果。

```
~$ ls /
bin    dev   initrd.img      lib64       mnt   root  snap   sys  var
boot   etc   initrd.img.old  lost+found  opt   run   sonic  tmp  vmlinuz
cdrom  home  lib             media       proc  sbin  srv    usr  vmlinuz.old
~$ echo $#
0
```
