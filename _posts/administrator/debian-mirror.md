title: 本地 mirror debian
date: 2018-01-10 19:57:20
toc: true
tags: [administrator, debian, mirror]
categories: administrator
keywords: [local, mirror, debian, debian-security, administrator, 本地, 镜像, jessie]
description: 本地镜像 debian 的记录。
---

## 需求

为快速编译，本地镜像 debian。

## 镜像工具

* ftpsync，debian 官方推荐
* apt-mirror，实测速率太慢
* debmirror，未测

一开始使用 apt-mirror，在 ubuntu 中 apt-get install 的 apt-mirror 并不能做 debian 镜像，出现各种提示错误。所以如果要用 apt-mirror，ubuntu 做 ubuntu 的，debian 上做 debian的。如果想在 ubuntu 上做 debian 镜像，那就搞一个 debian docker 吧。

## ftpsync 使用记录

官方对做 mirror 的说明，详见https://www.debian.org/mirror/ftpmirror。

ftpsync 的获取方式：

* tar 包 from https://ftp-master.debian.org/ftpsync.tar.gz
* git repository: git clone https://anonscm.debian.org/git/mirror/archvsync.git (see https://anonscm.debian.org/cgit/mirror/archvsync.git/)

使用说明见：https://anonscm.debian.org/cgit/mirror/archvsync.git/tree/README.md，相对简单。

快速入门：

* 建议创建一个特定的用户用于整个镜像源制作（我未做）
* 建议创建一个独立的目录用于整个镜像源制作，上一步创建的用户有写权限。（这一步肯定有必要）
* 将 ftpsync 脚本放在 $HOME/bin 或者 $HOME 下（我觉得这是为什么要创建特定用户的原因，保证这个目录只用于做镜像源用，其他用户也不会烦要在 /home/xxx 目录下有这种业务脚本）
* 将 ftpsync.conf.sample 重命名为 ftpsync.conf 放到 $HOME/etc 中，并根据你自己的需要做配置。至少你得改 `TO=` 和 `RSYNC_HOST` 两行。
  + `TO=` 指明下载的结果存放在本地哪个路径
  + `RSYNC_HOST` 指明上级镜像源，即你将从哪个服务器下载
* 创建 $HOME/log 目录，或者在 $LOGDIR 中指定
* 执行 ftpsync，等下载结束了！

还有另一处方式，由上级镜像源触发更新操作，则需要做其他处理：

> If only you receive an update trigger, Setup the .ssh/authorized_keys for the mirror user and place the public key of your upstream mirror into it. Preface it with `no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty,command="~/bin/ftpsync",from="IPADDRESS"` and replace $IPADDRESS with that of your upstream mirror.


arch 配置项，有两个互斥的选项，`ARCH_INCLUDE` 和 `ARCH_EXCLUDE`，相当于要下载的白名单和黑名单。查看 `bin/ftpsync` 脚本的源码，可以看出两个选项只能配置一个。建议直接配置 ARCH_INCLUDE，因为 ARCH_EXCLUDE 容易有漏网之鱼。

```
# Learn which archs to include/exclude based on ARCH_EXCLUDE and ARCH_INCLUDE
# settings.
# Sets EXCLUDE (which might also have --include statements
# followed by a --exclude *_*.<things>.
set_exclude_include_archs() {
    if [[ -n "${ARCH_EXCLUDE}" ]] && [[ -n "${ARCH_INCLUDE}" ]]; then
        echo >&2 "ARCH_EXCLUDE and ARCH_INCLUDE are mutually exclusive.  Set only one."
        exit 1
    fi
    
    if [[ -n "${ARCH_EXCLUDE}" ]]; then
        for ARCH in ${ARCH_EXCLUDE}; do
            arch_exclude ${ARCH}
        done
        arch_include '*'
        arch_include source
    elif [[ -n "${ARCH_INCLUDE}" ]]; then
        local include_arch_all=false 
        for ARCH in ${ARCH_INCLUDE}; do
            arch_include ${ARCH}
            if [[ ${ARCH} != source ]]; then
                include_arch_all=true
            fi
        done
        if [[ true = ${include_arch_all} ]]; then
            arch_include all
        fi
        arch_exclude '*'
        arch_exclude source
    fi
}
```

## 配置文件
### $HOME/etc/ftpsync.conf

```
########################################################################
########################################################################
## This is a sample configuration file for the ftpsync mirror script. ##
## Only options most users may need are included.  For documentation  ##
## and all available options see ftpsync.conf(5).                     ##
########################################################################
########################################################################

MIRRORNAME=`hostname -f`
TO="/home/mirror/mirror/debian"
MAILTO="$LOGNAME"
# HUB=false

########################################################################
## Connection options
########################################################################

RSYNC_HOST="ftp.cn.debian.org"
RSYNC_PATH="debian"
# RSYNC_USER=
# RSYNC_PASSWORD=

########################################################################
## Mirror information options
########################################################################

# INFO_MAINTAINER="Admins <admins@example.com>, Person <person@example.com>"
# INFO_SPONSOR="Example <https://example.com>"
# INFO_COUNTRY=DE
# INFO_LOCATION="Example"
# INFO_THROUGHPUT=10Gb

########################################################################
## Include and exclude options
########################################################################

ARCH_INCLUDE="amd64 source"
# ARCH_EXCLUDE=

########################################################################
## Log option
########################################################################

# LOGDIR=
```

### $HOME/etc/ftpsync-security.conf

此配置只下载 amd64 和 source，其他架构忽略。
RSYNC_HOST 尝试 `ftp.cn.debian.org` 和 `mirrors.ustc.edu.cn"`，都不行，只能用 `security.debian.org`。目前还没有搞懂，感觉很奇怪，这两个源都有 debian-security 目录。

```
########################################################################
########################################################################
## This is a sample configuration file for the ftpsync mirror script. ##
## Only options most users may need are included.  For documentation  ##
## and all available options see ftpsync.conf(5).                     ##
########################################################################
########################################################################

# MIRRORNAME=`hostname -f`
# TO="/srv/mirrors/debian/"
# MAILTO="$LOGNAME"
# HUB=false
MIRRORNAME=`hostname -f`
TO="/home/mirror/mirror/debian-security"
MAILTO="$LOGNAME"

########################################################################
## Connection options
########################################################################

RSYNC_HOST="security.debian.org"
RSYNC_PATH="debian-security"
# RSYNC_USER=
# RSYNC_PASSWORD=

########################################################################
## Mirror information options
########################################################################

# INFO_MAINTAINER="Admins <admins@example.com>, Person <person@example.com>"
# INFO_SPONSOR="Example <https://example.com>"
# INFO_COUNTRY=DE
# INFO_LOCATION="Example"
# INFO_THROUGHPUT=10Gb

########################################################################
## Include and exclude options
########################################################################

ARCH_INCLUDE="amd64 source"
# ARCH_EXCLUDE=

########################################################################
## Log option
########################################################################

# LOGDIR=
```

### 做 debian-security 镜像源

配置文件如上所示，执行的命令为 `./bin/ftpsync sync:archive:security`，注意字符串的一致，这里 security 要和配置文件 ftpsync-security.conf 中的 security 保持一致。

### cron 每天定时 sync

```
$ cat /etc/cron.d/ftpsync
SHELL=/bin/bash
PATH=/home/mirror/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

#minute hour day_of_month month day_of_week user command
0 2 * * * mirror ftpsync sync:archive:security
0 2 * * * mirror ftpsync
```

### log

如果下载失败，中间过程可见 LOGDIR 中的 log 进行排查。例如先前排查 debian-security 下载失败的问题。

* ftpsync.log，ftpsync 脚本的 log
* rsync-ftpsync.log，rsync 的正常 log，很多。。
* rsync-ftpsync.error，rsync 的异常 log，出问题看这里，可以看到 rsync 的 error code 及出现这个 error code 的原因。

```
$ ls log/
ftpsync.log    ftpsync-security.log    rsync-ftpsync.error.1  rsync-ftpsync.log.1             rsync-ftpsync-security.error.1
ftpsync.log.0  ftpsync-security.log.0  rsync-ftpsync.error.2  rsync-ftpsync.log.2             rsync-ftpsync-security.log
ftpsync.log.1  ftpsync-security.log.1  rsync-ftpsync.error.3  rsync-ftpsync.log.3             rsync-ftpsync-security.log.0
ftpsync.log.2  rsync-ftpsync.error     rsync-ftpsync.log      rsync-ftpsync-security.error    rsync-ftpsync-security.log.1
ftpsync.log.3  rsync-ftpsync.error.0   rsync-ftpsync.log.0    rsync-ftpsync-security.error.0

$ cat log/ftpsync-security.log.0 
1月 10 19:38:10 hostname ftpsync-security[12253]: Mirrorsync start
1月 10 19:38:10 hostname ftpsync-security[12253]: We got pushed from 10.13.70.86
1月 10 19:38:10 hostname ftpsync-security[12253]: Running mirrorsync, update is required, /home/mirror-user/mirrors/debian-security//Archive-Update-Required-hostname exists
1月 10 19:38:10 hostname ftpsync-security[12253]: Running stage1: rsync --filter=exclude_/Archive-Update-in-Progress-hostname --filter=protect_/Archive-Update-in-Progress-hostname --filter=exclude_/Archive-Update-Required-hostname --filter=protect_/Archive-Update-Required-hostname --filter=exclude_/project/trace/hostname --filter=protect_/project/trace/hostname --filter=exclude_/project/trace/hostname-stage1 --filter=protect_/project/trace/hostname-stage1 --filter=exclude_/project/trace/_hierarchy --filter=protect_/project/trace/_hierarchy --filter=exclude_/project/trace/_traces --filter=protect_/project/trace/_traces --filter=include_/project/ --filter=protect_/project/ --filter=include_/project/trace/ --filter=protect_/project/trace/ --filter=include_/project/trace/* --exclude=.~tmp~/ --filter=exclude_/dists/**/binary-alpha/ --filter=exclude_/dists/**/installer-alpha/ --filter=exclude_/dists/**/Contents-alpha.gz --filter=exclude_/dists/**/Contents-udeb-alpha.gz --filter=exclude_/dists/**/Contents-alpha.diff/ --filter=exclude_/indices/**/arch-alpha.files --filter=exclude_/indices/**/arch-alpha.list.gz --filter=exclude_/pool/**/*_alpha.deb --filter=exclude_/pool/**/*_alpha.udeb --filter=exclude_/pool/**/*_alpha.changes --filter=exclude_/dists/**/binary-arm/ --filter=exclude_/dists/**/installer-arm/ --filter=exclude_/dists/**/Contents-arm.gz --filter=exclude_/dists/**/Contents-udeb-arm.gz --filter=exclude_/dists/**/Contents-arm.diff/ --filter=exclude_/indices/**/arch-arm.files --filter=exclude_/indices/**/arch-arm.list.gz --filter=exclude_/pool/**/*_arm.deb --filter=exclude_/pool/**/*_arm.udeb --filter=exclude_/pool/**/*_arm.changes --filter=exclude_/dists/**/binary-armel/ --filter=exclude_/dists/**/installer-armel/ --filter=exclude_/dists/**/Contents-armel.gz --filter=exclude_/dists/**/Contents-udeb-armel.gz --filter=exclude_/dists/**/Contents-armel.diff/ --filter=exclude_/indices/**/arch-armel.files --filter=exclude_/indices/**/arch-armel.list.gz --filter=exclude_/pool/**/*_armel.deb --filter=exclude_/pool/**/*_armel.udeb --filter=exclude_/pool/**/*_armel.changes --filter=exclude_/dists/**/binary-armhf/ --filter=exclude_/dists/**/installer-armhf/ --filter=exclude_/dists/**/Contents-armhf.gz --filter=exclude_/dists/**/Contents-udeb-armhf.gz --filter=exclude_/dists/**/Contents-armhf.diff/ --filter=exclude_/indices/**/arch-armhf.files --filter=exclude_/indices/**/arch-armhf.list.gz --filter=exclude_/pool/**/*_armhf.deb --filter=exclude_/pool/**/*_armhf.udeb --filter=exclude_/pool/**/*_armhf.changes --filter=exclude_/dists/**/binary-hppa/ --filter=exclude_/dists/**/installer-hppa/ --filter=exclude_/dists/**/Contents-hppa.gz --filter=exclude_/dists/**/Contents-udeb-hppa.gz --filter=exclude_/dists/**/Contents-hppa.diff/ --filter=exclude_/indices/**/arch-hppa.files --filter=exclude_/indices/**/arch-hppa.list.gz --filter=exclude_/pool/**/*_hppa.deb --filter=exclude_/pool/**/*_hppa.udeb --filter=exclude_/pool/**/*_hppa.changes --filter=exclude_/dists/**/binary-hurd-i386/ --filter=exclude_/dists/**/installer-hurd-i386/ --filter=exclude_/dists/**/Contents-hurd-i386.gz --filter=exclude_/dists/**/Contents-udeb-hurd-i386.gz --filter=exclude_/dists/**/Contents-hurd-i386.diff/ --filter=exclude_/indices/**/arch-hurd-i386.files --filter=exclude_/indices/**/arch-hurd-i386.list.gz --filter=exclude_/pool/**/*_hurd-i386.deb --filter=exclude_/pool/**/*_hurd-i386.udeb --filter=exclude_/pool/**/*_hurd-i386.changes --filter=exclude_/dists/**/binary-i386/ --filter=exclude_/dists/**/installer-i386/ --filter=exclude_/dists/**/Contents-i386.gz --filter=exclude_/dists/**/Contents-udeb-i386.gz --filter=exclude_/dists/**/Contents-i386.diff/ --filter=exclude_/indices/**/arch-i386.files --filter=exclude_/indices/**/arch-i386.list.gz --filter=exclude_/pool/**/*_i386.deb --filter=exclude_/pool/**/*_i386.udeb --filter=exclude_/pool/**/*_i386.changes --filter=exclude_/dists/**/binary-ia64/ --filter=exclude_/dists/**/installer-ia64/ --filter=exclude_/dists/**/Contents-ia64.gz --filter=exclude_/dists/**/Contents-udeb-ia64.gz --filter=exclude_/dists/**/Contents-ia64.diff/ --filter=exclude_/indices/**/arch-ia64.files --filter=exclude_/indices/**/arch-ia64.list.gz --filter=exclude_/pool/**/*_ia64.deb --filter=exclude_/pool/**/*_ia64.udeb --filter=exclude_/pool/**/*_ia64.changes --filter=exclude_/dists/**/binary-kfreebsd-amd64/ --filter=exclude_/dists/**/installer-kfreebsd-amd64/ --filter=exclude_/dists/**/Contents-kfreebsd-amd64.gz --filter=exclude_/dists/**/Contents-udeb-kfreebsd-amd64.gz --filter=exclude_/dists/**/Contents-kfreebsd-amd64.diff/ --filter=exclude_/indices/**/arch-kfreebsd-amd64.files --filter=exclude_/indices/**/arch-kfreebsd-amd64.list.gz --filter=exclude_/pool/**/*_kfreebsd-amd64.deb --filter=exclude_/pool/**/*_kfreebsd-amd64.udeb --filter=exclude_/pool/**/*_kfreebsd-amd64.changes --filter=exclude_/dists/**/binary-kfreebsd-i386/ --filter=exclude_/dists/**/installer-kfreebsd-i386/ --filter=exclude_/dists/**/Contents-kfreebsd-i386.gz --filter=exclude_/dists/**/Contents-udeb-kfreebsd-i386.gz --filter=exclude_/dists/**/Contents-kfreebsd-i386.diff/ --filter=exclude_/indices/**/arch-kfreebsd-i386.files --filter=exclude_/indices/**/arch-kfreebsd-i386.list.gz --filter=exclude_/pool/**/*_kfreebsd-i386.deb --filter=exclude_/pool/**/*_kfreebsd-i386.udeb --filter=exclude_/pool/**/*_kfreebsd-i386.changes --filter=exclude_/dists/**/binary-m68k/ --filter=exclude_/dists/**/installer-m68k/ --filter=exclude_/dists/**/Contents-m68k.gz --filter=exclude_/dists/**/Contents-udeb-m68k.gz --filter=exclude_/dists/**/Contents-m68k.diff/ --filter=exclude_/indices/**/arch-m68k.files --filter=exclude_/indices/**/arch-m68k.list.gz --filter=exclude_/pool/**/*_m68k.deb --filter=exclude_/pool/**/*_m68k.udeb --filter=exclude_/pool/**/*_m68k.changes --filter=exclude_/dists/**/binary-mipsel/ --filter=exclude_/dists/**/installer-mipsel/ --filter=exclude_/dists/**/Contents-mipsel.gz --filter=exclude_/dists/**/Contents-udeb-mipsel.gz --filter=exclude_/dists/**/Contents-mipsel.diff/ --filter=exclude_/indices/**/arch-mipsel.files --filter=exclude_/indices/**/arch-mipsel.list.gz --filter=exclude_/pool/**/*_mipsel.deb --filter=exclude_/pool/**/*_mipsel.udeb --filter=exclude_/pool/**/*_mipsel.changes --filter=exclude_/dists/**/binary-mips/ --filter=exclude_/dists/**/installer-mips/ --filter=exclude_/dists/**/Contents-mips.gz --filter=exclude_/dists/**/Contents-udeb-mips.gz --filter=exclude_/dists/**/Contents-mips.diff/ --filter=exclude_/indices/**/arch-mips.files --filter=exclude_/indices/**/arch-mips.list.gz --filter=exclude_/pool/**/*_mips.deb --filter=exclude_/pool/**/*_mips.udeb --filter=exclude_/pool/**/*_mips.changes --filter=exclude_/dists/**/binary-powerpc/ --filter=exclude_/dists/**/installer-powerpc/ --filter=exclude_/dists/**/Contents-powerpc.gz --filter=exclude_/dists/**/Contents-udeb-powerpc.gz --filter=exclude_/dists/**/Contents-powerpc.diff/ --filter=exclude_/indices/**/arch-powerpc.files --filter=exclude_/indices/**/arch-powerpc.list.gz --filter=exclude_/pool/**/*_powerpc.deb --filter=exclude_/pool/**/*_powerpc.udeb --filter=exclude_/pool/**/*_powerpc.changes --filter=exclude_/dists/**/binary-s390/ --filter=exclude_/dists/**/installer-s390/ --filter=exclude_/dists/**/Contents-s390.gz --filter=exclude_/dists/**/Contents-udeb-s390.gz --filter=exclude_/dists/**/Contents-s390.diff/ --filter=exclude_/indices/**/arch-s390.files --filter=exclude_/indices/**/arch-s390.list.gz --filter=exclude_/pool/**/*_s390.deb --filter=exclude_/pool/**/*_s390.udeb --filter=exclude_/pool/**/*_s390.changes --filter=exclude_/dists/**/binary-s390x/ --filter=exclude_/dists/**/installer-s390x/ --filter=exclude_/dists/**/Contents-s390x.gz --filter=exclude_/dists/**/Contents-udeb-s390x.gz --filter=exclude_/dists/**/Contents-s390x.diff/ --filter=exclude_/indices/**/arch-s390x.files --filter=exclude_/indices/**/arch-s390x.list.gz --filter=exclude_/pool/**/*_s390x.deb --filter=exclude_/pool/**/*_s390x.udeb --filter=exclude_/pool/**/*_s390x.changes --filter=exclude_/dists/**/binary-sh/ --filter=exclude_/dists/**/installer-sh/ --filter=exclude_/dists/**/Contents-sh.gz --filter=exclude_/dists/**/Contents-udeb-sh.gz --filter=exclude_/dists/**/Contents-sh.diff/ --filter=exclude_/indices/**/arch-sh.files --filter=exclude_/indices/**/arch-sh.list.gz --filter=exclude_/pool/**/*_sh.deb --filter=exclude_/pool/**/*_sh.udeb --filter=exclude_/pool/**/*_sh.changes --filter=exclude_/dists/**/binary-sparc/ --filter=exclude_/dists/**/installer-sparc/ --filter=exclude_/dists/**/Contents-sparc.gz --filter=exclude_/dists/**/Contents-udeb-sparc.gz --filter=exclude_/dists/**/Contents-sparc.diff/ --filter=exclude_/indices/**/arch-sparc.files --filter=exclude_/indices/**/arch-sparc.list.gz --filter=exclude_/pool/**/*_sparc.deb --filter=exclude_/pool/**/*_sparc.udeb --filter=exclude_/pool/**/*_sparc.changes --filter=include_/dists/**/binary-*/ --filter=include_/dists/**/installer-*/ --filter=include_/dists/**/Contents-*.gz --filter=include_/dists/**/Contents-udeb-*.gz --filter=include_/dists/**/Contents-*.diff/ --filter=include_/indices/**/arch-*.files --filter=include_/indices/**/arch-*.list.gz --filter=include_/pool/**/*_*.deb --filter=include_/pool/**/*_*.udeb --filter=include_/pool/**/*_*.changes --filter=include_/dists/**/source/ --filter=include_/pool/**/*.tar.* --filter=include_/pool/**/*.diff.* --filter=include_/pool/**/*.dsc mirrors.ustc.edu.cn::debian-security /home/mirror-user/mirrors/debian-security/ --bwlimit=0 -prltvHSB8192 --safe-links --chmod=D755,F644 --timeout 3600 --stats --no-human-readable --include=*.diff/ --exclude=*.diff/Index --exclude=Packages* --exclude=Sources* --exclude=Release* --exclude=InRelease --include=i18n/by-hash --exclude=i18n/* --exclude=ls-lR*
1月 10 19:38:11 hostname ftpsync-security[12253]: Back from rsync with returncode 5
1月 10 19:38:11 hostname ftpsync-security[12253]: ERROR: Sync step 1 went wrong, got errorcode 5. Logfile: /home/mirror-user/log/ftpsync-security.log
1月 10 19:39:12 hostname ftpsync-security[12253]: Mirrorsync done

$ cat log/rsync-ftpsync-security.error.0
@ERROR: Unknown module 'debian-security'
rsync error: error starting client-server protocol (code 5) at main.c(1653) [Receiver=3.1.1]
```

### 镜像总大小

详见官方的镜像大小记录，每日更新，https://www.debian.org/mirror/size。

如果要减少镜像的大小，需要配置 `ARCH_INCLUDE=`，ARCH_EXCLUDE 要排除太多东西。

```
mirror$ du -hd 1
35G     ./debian-security
487G    ./debian
522G    .
```

## 设置本地 mirror

### 搭建局域网镜像源服务器
见 (http://www.cnblogs.com/beynol/p/nginx-simple-file-server.html) 该链已失效。。

1. 安装 nginx, `sudo apt-get install nginx`
2. 配置 `/etc/nginx/conf.d/files_server.conf`
3. 重启 nginx 服务，`sudo service nginx restart`

```
cat /etc/nginx/conf.d/files_server.conf

server {
    listen  80;
    server_name    10.162.250.250;
    charset utf-8;
    root /home/mirror/mirror;
    location / {
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }
}
```

### debian OS 中的 sources.list 配置

* sources.list

```
deb http://192.168.250.250/debian jessie main
deb http://192.168.250.250/debian jessie-updates main
deb http://192.168.250.250/debian-security jessie/updates main
```

* apt-get update

```
$ docker run -it debian:jessie bash
root@bc80cd6b79e9:/# cd /etc/apt/
root@bc80cd6b79e9:/etc/apt# rm sources.list
root@bc80cd6b79e9:/etc/apt# echo "deb http://192.168.250.250/debian jessie main" >> sources.list
root@bc80cd6b79e9:/etc/apt# echo "deb http://192.168.250.250/debian jessie-updates main" >> sources.list
root@bc80cd6b79e9:/etc/apt# echo "deb http://192.168.250.250/debian-security jessie/updates main" >> sources.list
root@bc80cd6b79e9:/etc/apt# cat sources.list
deb http://192.168.250.250/debian jessie main
deb http://192.168.250.250/debian jessie-updates main
deb http://192.168.250.250/debian-security jessie/updates main
root@bc80cd6b79e9:/etc/apt# apt-get update
Ign http://192.168.250.250 jessie InRelease
Get:1 http://192.168.250.250 jessie-updates InRelease [145 kB]
Get:2 http://192.168.250.250 jessie/updates InRelease [94.4 kB]
Get:3 http://192.168.250.250 jessie Release.gpg [2434 B]
Get:4 http://192.168.250.250 jessie Release [148 kB]
Get:5 http://192.168.250.250 jessie-updates/main amd64 Packages [23.1 kB]
Get:6 http://192.168.250.250 jessie/updates/main amd64 Packages [644 kB]
Get:7 http://192.168.250.250 jessie/main amd64 Packages [9064 kB]
Fetched 10.1 MB in 5s (1739 kB/s)
Reading package lists... Done
root@bc80cd6b79e9:/etc/apt#
```

## 问题记录

Update 失败。原因：ftpsync 可能不是一次性能能完成下载，需要下载比较久。

```
root@b58bb3484992:/# cat etc/apt/sources.list
deb http://192.168.250.250/debian stretch main
deb http://192.168.250.250/debian stretch-updates main
deb http://192.168.250.250/debian-security stretch/updates main

root@b58bb3484992:/# cat etc/apt/sources.list
deb http://192.168.250.250/debian stretch main
deb http://192.168.250.250/debian stretch-updates main
deb http://192.168.250.250/debian-security stretch/updates main
root@b58bb3484992:/# 
root@b58bb3484992:/# apt-get update
Ign:1 http://192.168.250.250/debian stretch InRelease
Ign:2 http://192.168.250.250/debian stretch-updates InRelease
Get:3 http://192.168.250.250/debian-security stretch/updates InRelease [63.0 kB]
Ign:4 http://192.168.250.250/debian stretch Release
Ign:5 http://192.168.250.250/debian stretch-updates Release
Ign:6 http://192.168.250.250/debian stretch/main all Packages
Ign:7 http://192.168.250.250/debian stretch/main amd64 Packages
Ign:8 http://192.168.250.250/debian stretch-updates/main amd64 Packages
Ign:9 http://192.168.250.250/debian stretch-updates/main all Packages
Ign:6 http://192.168.250.250/debian stretch/main all Packages
Ign:7 http://192.168.250.250/debian stretch/main amd64 Packages
Ign:8 http://192.168.250.250/debian stretch-updates/main amd64 Packages
Ign:9 http://192.168.250.250/debian stretch-updates/main all Packages
Ign:6 http://192.168.250.250/debian stretch/main all Packages
Ign:7 http://192.168.250.250/debian stretch/main amd64 Packages
Ign:8 http://192.168.250.250/debian stretch-updates/main amd64 Packages
Ign:9 http://192.168.250.250/debian stretch-updates/main all Packages
Ign:6 http://192.168.250.250/debian stretch/main all Packages
Ign:7 http://192.168.250.250/debian stretch/main amd64 Packages
Ign:8 http://192.168.250.250/debian stretch-updates/main amd64 Packages
Ign:9 http://192.168.250.250/debian stretch-updates/main all Packages
Ign:6 http://192.168.250.250/debian stretch/main all Packages
Ign:7 http://192.168.250.250/debian stretch/main amd64 Packages
Ign:8 http://192.168.250.250/debian stretch-updates/main amd64 Packages
Ign:9 http://192.168.250.250/debian stretch-updates/main all Packages
Ign:6 http://192.168.250.250/debian stretch/main all Packages
Err:7 http://192.168.250.250/debian stretch/main amd64 Packages
  404  Not Found
Err:8 http://192.168.250.250/debian stretch-updates/main amd64 Packages
  404  Not Found
Ign:9 http://192.168.250.250/debian stretch-updates/main all Packages
Get:10 http://192.168.250.250/debian-security stretch/updates/main amd64 Packages [333 kB]
Fetched 396 kB in 0s (1023 kB/s)    
Reading package lists... Done
W: The repository 'http://192.168.250.250/debian stretch Release' does not have a Release file.
N: Data from such a repository can't be authenticated and is therefore potentially dangerous to use.
N: See apt-secure(8) manpage for repository creation and user configuration details.
W: The repository 'http://192.168.250.250/debian stretch-updates Release' does not have a Release file.
N: Data from such a repository can't be authenticated and is therefore potentially dangerous to use.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: Failed to fetch http://192.168.250.250/debian/dists/stretch/main/binary-amd64/Packages  404  Not Found
E: Failed to fetch http://192.168.250.250/debian/dists/stretch-updates/main/binary-amd64/Packages  404  Not Found
E: Some index files failed to download. They have been ignored, or old ones used instead.
root@b58bb3484992:/# 
```
