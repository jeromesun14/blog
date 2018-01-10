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
* 创建 $HOME/log 目录，或者在 $LOGDIR 中指定
* 执行 ftpsync，等下载结束了！

还有另一处方式，由上级镜像源触发更新操作，则需要做其他处理：
> If only you receive an update trigger,
   Setup the .ssh/authorized_keys for the mirror user and place the public key of
   your upstream mirror into it. Preface it with
   `no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty,command="~/bin/ftpsync",from="IPADDRESS"`
   and replace $IPADDRESS with that of your upstream mirror.

## 配置文件
###
### 

## 做 debian-security 镜像源

## 问题记录
### 

