title: Linux命令行查看磁盘空间使用情况 - df
date: 2014-12-27 23:48:18
toc: true
tags: [Linux, 磁盘管理]
categories: shell
keywords: [linux, command, df]
description: Linux命令行查看目录及子目录大小。
---

`df - report file system disk space usage`，用于查看磁盘空间使用情况。

基本使用方法：`df -h`，以human readable的形式直观显示磁盘空间使用情况。

```
sunnogo@a3e420:~/github/hexo$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        92G   19G   69G  22% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
udev            2.0G  4.0K  2.0G   1% /dev
tmpfs           395M  1.4M  394M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            2.0G   25M  2.0G   2% /run/shm
none            100M   20K  100M   1% /run/user
/dev/sda5       184G   60G  115G  35% /home
```

<!--more-->

还可以通过`df -i`查看inode使用情况。

```
sunnogo@a3e420:~/github/hexo$ df -i
Filesystem       Inodes  IUsed    IFree IUse% Mounted on
/dev/sda1       6111232 362844  5748388    6% /
none             212221      2   212219    1% /sys/fs/cgroup
udev             205558    531   205027    1% /dev
tmpfs            212221    581   211640    1% /run
none             212221      3   212218    1% /run/lock
none             212221     51   212170    1% /run/shm
none             212221     17   212204    1% /run/user
/dev/sda5      12214272 738413 11475859    7% /home
```

