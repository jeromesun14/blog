title: Linux 定时执行备份任务
date: 2017-04-25 14:18:00
toc: true
tags: [Linux, ubuntu]
categories: linux
keywords: [ubuntu, cron, 定时执行, scheduled, task, commands, 备份]
description: 通过 cron 定时执行备份任务。
---

本地搭建一个 dokuwiki 服务器，考虑备份需求，每天备份服务器内容到另一台设备。

* 服务器 IP：172.168.111.192
* 备份设备 IP：192.168.204.168

通过 cron 命令进行定时任务执行。

## 前期工作

见本博客另一篇文章 `scp 命令不需要密码`。

## 实现备份的脚本

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

每天 4:30 执行备份脚本。

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
