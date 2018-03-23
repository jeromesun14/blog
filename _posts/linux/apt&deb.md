title: apt&deb 工具使用记录
date: 2018-01-15 15:23:00
toc: true
tags: [ubuntu, deb]
categories: linux
keywords: [ubuntu, linux, deb, package, apt, 查看内容]
description: apt 和 deb 相关工具使用记录。
---

## 查看 deb 包内容

* 查看 .deb 文件包含的内容，`dpkg-deb -c packageName.deb`
* 查看已安装 deb 包的内容，`dpkg -L packageName`

## apt 卸载命令

http://blog.csdn.net/get_set/article/details/51276609

apt-get purge / apt-get --purge remove 
删除已安装包（不保留配置文件)。 
如软件包a，依赖软件包b，则执行该命令会删除a，而且不保留配置文件

apt-get autoremove 
删除为了满足依赖而安装的，但现在不再需要的软件包（包括已安装包），保留配置文件。

apt-get remove 
删除已安装的软件包（保留配置文件），不会删除依赖软件包，且保留配置文件。

apt-get autoclean 
APT的底层包是dpkg, 而dpkg 安装Package时, 会将 *.deb 放在 /var/cache/apt/archives/中，apt-get autoclean 只会删除 /var/cache/apt/archives/ 已经过期的deb。

apt-get clean 
使用 apt-get clean 会将 /var/cache/apt/archives/ 的 所有 deb 删掉，可以理解为 rm /var/cache/apt/archives/*.deb。

## apt-get install -y
对所有的提示执行 y（yes）.

## install 和 build-dep 差异
The short version.

apt-get install
installs a new package, automatically resolving and downloading dependent packages. If package is installed then try to upgrade to latest version.

apt-get build-dep
Causes apt-get to install/remove packages in an attempt to satisfy the build dependencies for a source package.

The command sudo apt-get build-dep packagename means to install all dependencies for 'packagename' so that I can build it". So build-dep is an apt-get command just like install, remove, update, etc.

The build-dep command searches the local repositories in the system and install the build dependencies for package. If the package does not exists in the local repository it will return an error code.

For installing matplotlib see To Install matplotlib on Ubuntu
