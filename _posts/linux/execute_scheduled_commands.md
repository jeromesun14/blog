title: Linux 定时执行备份任务
date: 2017-04-25 14:18:00
toc: true
tags: [Linux, ubuntu]
categories: linux
keywords: [ubuntu, cron, 定期执行, 任务, scheduled, task, commands, 备份, backup, dokuwiki, EDITOR, vim, nano]
description: 通过 cron 定时执行备份任务。
---

本地搭建一个 dokuwiki 服务器，考虑备份需求，每天备份服务器内容到另一台设备。

* 服务器 IP：172.168.111.192
* 备份设备 IP：192.168.204.168

通过 cron 命令进行定时任务执行。

## 前期工作

见本博客另一篇文章 [如何让 scp / ssh 不需要密码](sunyongfeng.com/201704/linux/scp_ssh_without_password.html)。

## 实现备份的脚本

在服务器 172.168.111.192 上实现备份脚本。

```
root@ubuntu:~# cat dokuwiki.backup.sh 
#!/bin/bash

backpath=/var/www/backup
date=`date +%y%m%d`
site='dokuwiki'
tar zcf ${backpath}${site}"-"${date}.tar.gz /var/www/${site}
scp ${backpath}${site}"-"${date}.tar.gz sunyongfeng@192.168.204.168:/home/sunyongfeng/backup/${site}"-"${date}.tar.gz
root@ubuntu:~# 
```

## cron 配置

在服务器 172.168.111192 上每天 4:30 执行备份脚本。

```
root@ubuntu:~# export EDITOR=vim 
root@ubuntu:~# crontab -e       
no crontab for root - using an empty one
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
30 04 * * * /root/dokuwiki.backup.sh
```

首次选择 cron 编辑器不小心选择了 nano，不懂怎么用。通过 `export EDITOR=vim` 将编辑器切成 vim。

online cron 表达式生成器: https://crontab-generator.org/

注意，每 30 分钟执行一次的写法：

* `*/30 * * * * ls >/dev/null 2>&1`
* 

## 效果

备份机器 192.168.204.168 上的效果：

```
sunyongfeng@openswitch-OptiPlex-380:~/backup$ ls -al
total 8759824
drwxrwxrwx   7 sunyongfeng sunyongfeng       4096 5月  31 04:30 .
drwxr-xr-x  44 sunyongfeng sunyongfeng       4096 5月  31 09:24 ..
-rw-r--r--   1 sunyongfeng sunyongfeng   30790523 4月  18 20:26 dokuwiki-170418.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30829080 4月  25 14:14 dokuwiki-170425.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30829128 4月  26 04:31 dokuwiki-170426.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30840695 4月  27 04:31 dokuwiki-170427.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30878801 4月  28 04:31 dokuwiki-170428.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30878561 4月  29 04:31 dokuwiki-170429.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30878561 4月  30 04:31 dokuwiki-170430.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30878561 5月   1 04:31 dokuwiki-170501.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30878561 5月   2 04:31 dokuwiki-170502.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   30925192 5月   3 04:31 dokuwiki-170503.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31827534 5月   4 04:31 dokuwiki-170504.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31830576 5月   5 04:31 dokuwiki-170505.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31830576 5月   6 04:31 dokuwiki-170506.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31830576 5月   7 04:32 dokuwiki-170507.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31830576 5月   8 04:31 dokuwiki-170508.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832468 5月   9 04:31 dokuwiki-170509.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832468 5月  10 04:31 dokuwiki-170510.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31833486 5月  11 04:31 dokuwiki-170511.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31834472 5月  12 04:31 dokuwiki-170512.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31833802 5月  13 04:31 dokuwiki-170513.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31833802 5月  14 04:31 dokuwiki-170514.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31835188 5月  15 04:31 dokuwiki-170515.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31835398 5月  16 04:30 dokuwiki-170516.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31834919 5月  17 04:30 dokuwiki-170517.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31834919 5月  18 04:30 dokuwiki-170518.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832950 5月  19 04:30 dokuwiki-170519.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832950 5月  20 04:30 dokuwiki-170520.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832950 5月  21 04:30 dokuwiki-170521.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832950 5月  22 04:30 dokuwiki-170522.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832950 5月  23 04:30 dokuwiki-170523.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31832950 5月  24 04:30 dokuwiki-170524.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31833337 5月  25 04:30 dokuwiki-170525.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31833232 5月  26 04:30 dokuwiki-170526.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31833446 5月  27 04:30 dokuwiki-170527.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31833446 5月  28 04:30 dokuwiki-170528.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31835384 5月  29 04:30 dokuwiki-170529.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31835403 5月  30 04:30 dokuwiki-170530.tar.gz
-rw-r--r--   1 sunyongfeng sunyongfeng   31835403 5月  31 04:30 dokuwiki-170531.tar.gz
```

## misc

* cron 的默认工作目录，`$HOME`

