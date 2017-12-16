title: apt-deb 相关问题记录
date: 2017-12-14 10:42:00
toc: true
tags: [ubuntu, deb, apt]
categories: linux
keywords: [ubuntu, linux, deb, apt, package, control, dependency, dependencies, build dependency, source.list, build-dep, backports, 编译依赖, 查看, ]
description: 记录 apt / deb 使用过程中的各种问题及解决方法。
---

##  修改源

## 查看依赖关系

* 编译依赖，`apt-rdepends --build-depends package_name`

## 问题记录
### apt-get -y build-dep linux 失败

log：

```
root@c60c282d2d76:/# apt-get -y build-dep linux
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Build-Depends dependency for linux cannot be satisfied because candidate version of package debhelper can't satisfy version requirements
```

分析：debhelper 的版本满足不了编译 linux 的依赖。

* 通过 `apt-rdepends --build-depends linux > linux.build-depends.txt` 查看 linux 的编译依赖。linux 的编译依赖好多，要等很长时间。
* 通过 `cat linux.build-depends.txt | grep debhelper` 确认对 debhelper 的版本依赖发现有软件对其的版本依赖 >= 10，而 debian jessie 的默认 debhelper 版本为 9.20xxx，在 repo 中可以看到 debhelper 当前最新版本为 10.10.9。

```
root@c60c282d2d76:/# cat linux.build-depends.txt | grep ">= 10"  
  Build-Depends: debhelper (>= 10~)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10~)
  Build-Depends: debhelper (>= 10~)
  Build-Depends: debhelper (>= 10)
  Build-Depends: debhelper (>= 10)
```

* 查看 debhelper 所有版本

通过 `apt-cache madison debhelper` 查看 debhelper 的所有版本

```
root@c60c282d2d76:/# apt-cache madison debhelper
 debhelper | 10.2.5~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main amd64 Packages
 debhelper | 9.20150101+deb8u2 | http://ftp.cn.debian.org/debian/ jessie/main amd64 Packages
 debhelper | 9.20150101+deb8u2 | http://security.debian.org/ jessie/updates/main amd64 Packages
 debhelper | 9.20150101+deb8u2 | http://ftp.cn.debian.org/debian/ jessie/main Sources
 debhelper | 10.2.5~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main Sources
 debhelper | 9.20150101+deb8u2 | http://security.debian.org/ jessie/updates/main Sources
```

尝试安装 debhelper 版本 10.2.5~bp08+1，发现有新的依赖关系问题：

```
root@c60c282d2d76:/# apt-get install debhelper=10.2.5~bpo8+1
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 debhelper : Depends: dh-autoreconf (>= 12~) but 10 is to be installed
             Depends: dh-strip-nondeterminism (>= 0.028~) but 0.003-1 is to be installed
E: Unable to correct problems, you have held broken packages.
root@c60c282d2d76:/# 
```

发现比较高版本的包都是来自 jessie-backports:

> [Backports](https://wiki.debian.org/Backports): Backports are recompiled packages from testing (mostly) and unstable (in a few cases only, e.g. security updates), so they will run without new libraries (wherever it is possible) on a stable Debian distribution. It is recommended to pick out single backports which fit your needs, and not to use all backports available.

即，Backports 存放的软件为不稳定或仅用于测试目的的编译结果。

```
root@c60c282d2d76:/# apt-cache madison dh-autoreconf        
dh-autoreconf |  12~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main amd64 Packages
dh-autoreconf |         10 | http://ftp.cn.debian.org/debian/ jessie/main amd64 Packages
dh-autoreconf |         10 | http://ftp.cn.debian.org/debian/ jessie/main Sources
dh-autoreconf |  12~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main Sources
root@c60c282d2d76:/# apt-cache madison dh-strip-nondeterminism 
dh-strip-nondeterminism | 0.034-1~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main amd64 Packages
dh-strip-nondeterminism |    0.003-1 | http://ftp.cn.debian.org/debian/ jessie/main amd64 Packages
strip-nondeterminism |    0.003-1 | http://ftp.cn.debian.org/debian/ jessie/main Sources
strip-nondeterminism | 0.034-1~bpo8+1 | http://ftp.cn.debian.org/debian/ jessie-backports/main Sources
root@c60c282d2d76:/# 
```

OK，是时候把 Backports 的源踢掉了。

故障时的源，被 [163 的 Debian镜像使用帮助](http://mirrors.163.com/.help/debian.html)误导了。

```
deb http://mirrors.163.com/debian/ jessie main non-free contrib
deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
```

在调试的时候，一开始已注意到可能是源的问题，但是还是一遍一遍傻逼式地试不同的源，比如上面的 debian 官方中国源 ftp.cn.debian.org。

**烂习惯。。。。。调东西还是乱试一通，没去找根本原因。。。**
教训是一个下午没了，正常已经半个小时就可以发现根本原因了。
