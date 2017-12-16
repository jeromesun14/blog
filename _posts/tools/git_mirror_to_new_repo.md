title: Mirror 旧 git 仓库到新的 git 仓库（保留所有提交 / 分支 / tags）
date: 2017-12-16 16:32:30
toc: true
tags: [版本控制, git]
categories: tools
keywords: [git, command, version control, branch, keep, all commits, branches, tags]
description: git 镜像已有仓库到 gitlab，保留所有的提交、分支和 tags。
---

## 需求

感谢 GFW，在编译 SONiC 开源项目的时候碰到很多源码下载不下来。因此有必要本地保存一份源码。
而且，有的源码下载下来，需要切换分支，因此需要完整保存该源码的镜像，包含所有的提交、分支和 tags。

本文以自建的 gitlab ce 服务器为例。目标：mirror https://anonscm.debian.org/cgit/pkg-dhcp/isc-dhcp.git 到本地的 gitlab 服务器。

## 方法一：命令行

很明显，这是一个很常见的需求，git 可以很快地做这个活。详见 [Import an existing git project into GitLab?](https://stackoverflow.com/questions/20359936/import-an-existing-git-project-into-gitlab)

* 在 gitlab ce 服务器上添加对应的仓库，比如 `http://your_gitlab_url/sonic/isc-dhcp.git`
* 下载 isc-dhcp 的源码到本地，`git clone https://anonscm.debian.org/cgit/pkg-dhcp/isc-dhcp.git --mirror`
* 进入 isc-dhcp 源码根目录，执行以下两个命令：
  + git remote add gitlab http://your_gitlab_url/sonic/isc-dhcp.git
  + git push gitlab --mirror
* Bingo!!

```
netadmin@kmc-b0230:~$ git clone https://anonscm.debian.org/cgit/pkg-dhcp/isc-dhcp.git --mirror
Cloning into bare repository 'isc-dhcp.git'...
remote: Counting objects: 3435, done.
remote: Compressing objects: 100% (1702/1702), done.
remote: Total 3435 (delta 2428), reused 2488 (delta 1670)
Receiving objects: 100% (3435/3435), 64.05 MiB | 62.00 KiB/s, done.
Resolving deltas: 100% (2428/2428), done.
Checking connectivity... done.
netadmin@kmc-b0230:~/isc-dhcp$ git remote add gitlab http://your_gitlab_url/sonic/isc-dhcp.git
netadmin@kmc-b0230:~/isc-dhcp$ git push gitlab --mirror
Username for 'http://your_gitlab_url': jeromesun
Password for 'http://jeromesun@your_gitlab_url':
Counting objects: 3435, done.
Delta compression using up to 24 threads.
Compressing objects: 100% (944/944), done.
Writing objects: 100% (3435/3435), 64.05 MiB | 79.66 MiB/s, done.
Total 3435 (delta 2428), reused 3435 (delta 2428)
remote: Resolving deltas: 100% (2428/2428), done.
To http://your_gitlab_url/sonic/isc-dhcp.git
 * [new branch]      experimental -> experimental
 * [new branch]      master -> master
 * [new branch]      pristine-tar -> pristine-tar
 * [new branch]      upstream -> upstream
 * [new tag]         debian/4.1.0-1 -> debian/4.1.0-1
 * [new tag]         debian/4.1.1-1 -> debian/4.1.1-1
 * [new tag]         debian/4.1.1-2 -> debian/4.1.1-2
 * [new tag]         debian/4.1.1-P1-10 -> debian/4.1.1-P1-10
 * [new tag]         debian/4.1.1-P1-11 -> debian/4.1.1-P1-11
 * [new tag]         debian/4.1.1-P1-12 -> debian/4.1.1-P1-12
 * [new tag]         debian/4.1.1-P1-13 -> debian/4.1.1-P1-13
 * [new tag]         debian/4.1.1-P1-14 -> debian/4.1.1-P1-14
 * [new tag]         debian/4.1.1-P1-15 -> debian/4.1.1-P1-15
 * [new tag]         debian/4.1.1-P1-16 -> debian/4.1.1-P1-16
 * [new tag]         debian/4.1.1-P1-16.1 -> debian/4.1.1-P1-16.1
 * [new tag]         debian/4.1.1-P1-17 -> debian/4.1.1-P1-17
 * [new tag]         debian/4.1.1-P1-4 -> debian/4.1.1-P1-4
 * [new tag]         debian/4.1.1-P1-5 -> debian/4.1.1-P1-5
 * [new tag]         debian/4.1.1-P1-6 -> debian/4.1.1-P1-6
 * [new tag]         debian/4.1.1-P1-7 -> debian/4.1.1-P1-7
 * [new tag]         debian/4.1.1-P1-8 -> debian/4.1.1-P1-8
 * [new tag]         debian/4.1.1-P1-9 -> debian/4.1.1-P1-9
 * [new tag]         debian/4.2.2-1 -> debian/4.2.2-1
 * [new tag]         debian/4.2.2-2 -> debian/4.2.2-2
 * [new tag]         debian/4.2.2.dfsg.1-3 -> debian/4.2.2.dfsg.1-3
 * [new tag]         debian/4.2.2.dfsg.1-4 -> debian/4.2.2.dfsg.1-4
 * [new tag]         debian/4.2.2.dfsg.1-5 -> debian/4.2.2.dfsg.1-5
 * [new tag]         debian/4.2.4-1 -> debian/4.2.4-1
 * [new tag]         debian/4.2.4-2 -> debian/4.2.4-2
 * [new tag]         debian/4.2.4-3 -> debian/4.2.4-3
 * [new tag]         debian/4.2.4-4 -> debian/4.2.4-4
 * [new tag]         debian/4.2.4-5 -> debian/4.2.4-5
 * [new tag]         debian/4.2.4-6 -> debian/4.2.4-6
 * [new tag]         debian/4.2.4-7 -> debian/4.2.4-7
 * [new tag]         debian/4.3.0+dfsg-1 -> debian/4.3.0+dfsg-1
 * [new tag]         debian/4.3.0+dfsg-2 -> debian/4.3.0+dfsg-2
 * [new tag]         debian/4.3.0a1-1 -> debian/4.3.0a1-1
 * [new tag]         debian/4.3.0a1-2 -> debian/4.3.0a1-2
 * [new tag]         debian/4.3.1-1 -> debian/4.3.1-1
 * [new tag]         debian/4.3.1-2 -> debian/4.3.1-2
 * [new tag]         debian/4.3.1-3 -> debian/4.3.1-3
 * [new tag]         debian/4.3.1-4 -> debian/4.3.1-4
 * [new tag]         debian/4.3.1-5 -> debian/4.3.1-5
 * [new tag]         debian/4.3.1-6 -> debian/4.3.1-6
 * [new tag]         debian/4.3.1_b1 -> debian/4.3.1_b1
 * [new tag]         debian/4.3.2-1 -> debian/4.3.2-1
 * [new tag]         debian/4.3.3-1 -> debian/4.3.3-1
 * [new tag]         debian/4.3.3-2 -> debian/4.3.3-2
 * [new tag]         debian/4.3.3-3 -> debian/4.3.3-3
 * [new tag]         debian/4.3.3-5 -> debian/4.3.3-5
 * [new tag]         debian/4.3.3-6 -> debian/4.3.3-6
 * [new tag]         debian/4.3.3-7 -> debian/4.3.3-7
 * [new tag]         debian/4.3.3-8 -> debian/4.3.3-8
 * [new tag]         debian/4.3.3-9 -> debian/4.3.3-9
 * [new tag]         debian/4.3.4-1 -> debian/4.3.4-1
 * [new tag]         debian/4.3.5-1 -> debian/4.3.5-1
 * [new tag]         debian/4.3.5-2 -> debian/4.3.5-2
 * [new tag]         debian/4.3.5-3 -> debian/4.3.5-3
 * [new tag]         debian/4.3.5_b1-1 -> debian/4.3.5_b1-1
 * [new tag]         debin/4.2.2.dfsg.1-4 -> debin/4.2.2.dfsg.1-4
 * [new tag]         upstream/4.1.0 -> upstream/4.1.0
 * [new tag]         upstream/4.1.1 -> upstream/4.1.1
 * [new tag]         upstream/4.1.1-P1 -> upstream/4.1.1-P1
 * [new tag]         upstream/4.2.2 -> upstream/4.2.2
 * [new tag]         upstream/4.2.4 -> upstream/4.2.4
 * [new tag]         upstream/4.2.5-P1 -> upstream/4.2.5-P1
 * [new tag]         upstream/4.3.0 -> upstream/4.3.0
 * [new tag]         upstream/4.3.0a1 -> upstream/4.3.0a1
 * [new tag]         upstream/4.3.1 -> upstream/4.3.1
 * [new tag]         upstream/4.3.2 -> upstream/4.3.2
 * [new tag]         upstream/4.3.3 -> upstream/4.3.3
 * [new tag]         upstream/4.3.4 -> upstream/4.3.4
 * [new tag]         upstream/4.3.4_b1 -> upstream/4.3.4_b1
 * [new tag]         upstream/4.3.5 -> upstream/4.3.5
 * [new tag]         upstream/4.3.5_b1 -> upstream/4.3.5_b1
netadmin@kmc-b0230:~/isc-dhcp$
```

## 方法二：gitlab 使用 import project 创建新项目

详见 gitlab 图形界面，支持从 github / bitbucket / google code 等网站直接 import，也支持通过 URL 指定要 import 的 git 仓库。

![git-mirror](/images/tools/git/git-mirror.png)
