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

## 查看包是否已安装

* `dpkg -s packageName`

## 解压 deb 包

* `dpkg -x xxx.deb your_dir`

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

## 清除多余的 kernel image

问题：apt-get install 的时候出错，提示 `update-initramfs` 做 gzip 时没有空间。根本原因为 `/boot` 分区没空间。
解决：通过 `apt-get remove --purge` 清除多余的 kernel image。

```
$ df -h
Filesystem                   Size  Used Avail Use% Mounted on
udev                          63G     0   63G   0% /dev
tmpfs                         13G   34M   13G   1% /run
/dev/mapper/ubuntu--vg-root  1.6T  392G  1.2T  26% /
tmpfs                         63G  176K   63G   1% /dev/shm
tmpfs                        5.0M     0  5.0M   0% /run/lock
tmpfs                         63G     0   63G   0% /sys/fs/cgroup
/dev/sda1                    472M  404M   44M  91% /boot

~$ apt-file search arp
The program 'apt-file' is currently not installed. You can install it by typing:
sudo apt install apt-file
netadmin@kmc-b0230:~/sb.0509$ sudo apt-get install apt-file
[sudo] password for netadmin: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-headers-4.13.0-37 linux-headers-4.13.0-37-generic linux-headers-4.13.0-38 linux-headers-4.13.0-38-generic linux-headers-4.13.0-39 linux-headers-4.13.0-39-generic linux-image-4.13.0-37-generic
  linux-image-4.13.0-38-generic linux-image-4.13.0-39-generic linux-image-4.4.0-119-generic linux-image-4.4.0-121-generic linux-image-4.4.0-124-generic linux-image-extra-4.13.0-37-generic
  linux-image-extra-4.13.0-38-generic linux-image-extra-4.13.0-39-generic linux-image-extra-4.4.0-119-generic linux-image-extra-4.4.0-121-generic linux-image-extra-4.4.0-124-generic
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libconfig-file-perl libregexp-assemble-perl
The following NEW packages will be installed:
  apt-file libconfig-file-perl libregexp-assemble-perl
0 upgraded, 3 newly installed, 0 to remove and 212 not upgraded.
13 not fully installed or removed.
Need to get 109 kB of archives.
After this operation, 349 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://cn.archive.ubuntu.com/ubuntu xenial/universe amd64 libconfig-file-perl all 1.50-3 [9,722 B]
Get:2 http://cn.archive.ubuntu.com/ubuntu xenial/universe amd64 libregexp-assemble-perl all 0.36-1 [77.5 kB]
Get:3 http://cn.archive.ubuntu.com/ubuntu xenial/universe amd64 apt-file all 2.5.5ubuntu1 [21.6 kB]
Fetched 109 kB in 2s (42.7 kB/s)     
Selecting previously unselected package libconfig-file-perl.
(Reading database ... 382259 files and directories currently installed.)
Preparing to unpack .../libconfig-file-perl_1.50-3_all.deb ...
Unpacking libconfig-file-perl (1.50-3) ...
Selecting previously unselected package libregexp-assemble-perl.
Preparing to unpack .../libregexp-assemble-perl_0.36-1_all.deb ...
Unpacking libregexp-assemble-perl (0.36-1) ...
Selecting previously unselected package apt-file.
Preparing to unpack .../apt-file_2.5.5ubuntu1_all.deb ...
Unpacking apt-file (2.5.5ubuntu1) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up initramfs-tools (0.122ubuntu8.8) ...
update-initramfs: deferring update (trigger activated)
Setting up linux-firmware (1.157.18) ...
update-initramfs: Generating /boot/initrd.img-4.13.0-41-generic


gzip: stdout: No space left on device
E: mkinitramfs failure find 141 cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-4.13.0-41-generic with 1.
dpkg: error processing package linux-firmware (--configure):
 subprocess installed post-installation script returned error exit status 1
Setting up linux-image-4.13.0-43-generic (4.13.0-43.48~16.04.1) ...
Running depmod.
update-initramfs: deferring update (hook will be called later)
The link /initrd.img is a dangling linkto /boot/initrd.img-4.4.0-127-generic
Examining /etc/kernel/postinst.d.
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 4.13.0-43-generic /boot/vmlinuz-4.13.0-43-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 4.13.0-43-generic /boot/vmlinuz-4.13.0-43-generic
update-initramfs: Generating /boot/initrd.img-4.13.0-43-generic

gzip: stdout: No space left on device
E: mkinitramfs failure find 141 cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-4.13.0-43-generic with 1.
run-parts: /etc/kernel/postinst.d/initramfs-tools exited with return code 1
Failed to process /etc/kernel/postinst.d at /var/lib/dpkg/info/linux-image-4.13.0-43-generic.postinst line 1052.
dpkg: error processing package linux-image-4.13.0-43-generic (--configure):
 subprocess installed post-installation script returned error exit status 2
dpkg: dependency problems prevent configuration of linux-image-extra-4.13.0-43-generic:
 linux-image-extra-4.13.0-43-generic depends on linux-image-4.13.0-43-generic; however:
  Package linux-image-4.13.0-43-generic is not configured yet.

dpkg: error processing package linux-image-extra-4.13.0-43-generic (--configure):
 dependency problems - leaving unconfigured
No apport report written because the error message indicates its a followup error from a previous failure.
                                                                                                          dpkg: dependency problems prevent configuration of linux-image-generic-hwe-16.04:
 linux-image-generic-hwe-16.04 depends on linux-image-4.13.0-43-generic; however:
  Package linux-image-4.13.0-43-generic is not configured yet.
 linux-image-generic-hwe-16.04 depends on linux-image-extra-4.13.0-43-generic; however:
  Package linux-image-extra-4.13.0-43-generic is not configured yet.
 linux-image-generic-hwe-16.04 depends on linux-firmware; however:
  Package linux-firmware is not configured yet.

dpkg: error processing package linux-image-generic-hwe-16.04 (--configure):
 dependency problems - leaving unconfigured
No apport report written because MaxReports is reached already
                                                              dpkg: dependency problems prevent configuration of linux-generic-hwe-16.04:
 linux-generic-hwe-16.04 depends on linux-image-generic-hwe-16.04 (= 4.13.0.43.62); however:
  Package linux-image-generic-hwe-16.04 is not configured yet.

dpkg: error processing package linux-generic-hwe-16.04 (--configure):
 dependency problems - leaving unconfigured
No apport report written because MaxReports is reached already
                                                              Setting up linux-image-4.4.0-124-generic (4.4.0-124.148) ...
Running depmod.
update-initramfs: deferring update (hook will be called later)
The link /initrd.img is a dangling linkto /boot/initrd.img-4.13.0-43-generic
Examining /etc/kernel/postinst.d.
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 4.4.0-124-generic /boot/vmlinuz-4.4.0-124-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 4.4.0-124-generic /boot/vmlinuz-4.4.0-124-generic
update-initramfs: Generating /boot/initrd.img-4.4.0-124-generic

gzip: stdout: No space left on device
E: mkinitramfs failure find 141 cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-4.4.0-124-generic with 1.
run-parts: /etc/kernel/postinst.d/initramfs-tools exited with return code 1
Failed to process /etc/kernel/postinst.d at /var/lib/dpkg/info/linux-image-4.4.0-124-generic.postinst line 1052.
dpkg: error processing package linux-image-4.4.0-124-generic (--configure):
 subprocess installed post-installation script returned error exit status 2
No apport report written because MaxReports is reached already
                                                              Setting up linux-image-4.4.0-127-generic (4.4.0-127.153) ...
Running depmod.
update-initramfs: deferring update (hook will be called later)
The link /initrd.img is a dangling linkto /boot/initrd.img-4.4.0-124-generic
Examining /etc/kernel/postinst.d.
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
update-initramfs: Generating /boot/initrd.img-4.4.0-127-generic

gzip: stdout: No space left on device
E: mkinitramfs failure find 141 cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-4.4.0-127-generic with 1.
run-parts: /etc/kernel/postinst.d/initramfs-tools exited with return code 1
Failed to process /etc/kernel/postinst.d at /var/lib/dpkg/info/linux-image-4.4.0-127-generic.postinst line 1052.
dpkg: error processing package linux-image-4.4.0-127-generic (--configure):
 subprocess installed post-installation script returned error exit status 2
No apport report written because MaxReports is reached already
                                                              Setting up linux-image-extra-4.13.0-41-generic (4.13.0-41.46~16.04.1) ...
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 4.13.0-41-generic /boot/vmlinuz-4.13.0-41-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 4.13.0-41-generic /boot/vmlinuz-4.13.0-41-generic
update-initramfs: Generating /boot/initrd.img-4.13.0-41-generic

 
gzip: stdout: No space left on device
E: mkinitramfs failure find 141 cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-4.13.0-41-generic with 1.
run-parts: /etc/kernel/postinst.d/initramfs-tools exited with return code 1
dpkg: error processing package linux-image-extra-4.13.0-41-generic (--configure):
 subprocess installed post-installation script returned error exit status 1
No apport report written because MaxReports is reached already
                                                              dpkg: dependency problems prevent configuration of linux-image-extra-4.4.0-124-generic:
 linux-image-extra-4.4.0-124-generic depends on linux-image-4.4.0-124-generic; however:
  Package linux-image-4.4.0-124-generic is not configured yet.

dpkg: error processing package linux-image-extra-4.4.0-124-generic (--configure):
 dependency problems - leaving unconfigured
No apport report written because MaxReports is reached already
                                                              dpkg: dependency problems prevent configuration of linux-image-extra-4.4.0-127-generic:
 linux-image-extra-4.4.0-127-generic depends on linux-image-4.4.0-127-generic; however:
  Package linux-image-4.4.0-127-generic is not configured yet.

dpkg: error processing package linux-image-extra-4.4.0-127-generic (--configure):
 dependency problems - leaving unconfigured
No apport report written because MaxReports is reached already
                                                              dpkg: dependency problems prevent configuration of linux-image-generic:
 linux-image-generic depends on linux-image-4.4.0-127-generic; however:
  Package linux-image-4.4.0-127-generic is not configured yet.
 linux-image-generic depends on linux-image-extra-4.4.0-127-generic; however:
  Package linux-image-extra-4.4.0-127-generic is not configured yet.
 linux-image-generic depends on linux-firmware; however:
  Package linux-firmware is not configured yet.

dpkg: error processing package linux-image-generic (--configure):
 dependency problems - leaving unconfigured
No apport report written because MaxReports is reached already
                                                              dpkg: dependency problems prevent configuration of linux-image-extra-virtual:
 linux-image-extra-virtual depends on linux-image-generic (= 4.4.0.127.133); however:
  Package linux-image-generic is not configured yet.

dpkg: error processing package linux-image-extra-virtual (--configure):
 dependency problems - leaving unconfigured
No apport report written because MaxReports is reached already
                                                              Setting up libconfig-file-perl (1.50-3) ...
Setting up libregexp-assemble-perl (0.36-1) ...
Setting up apt-file (2.5.5ubuntu1) ...
The system-wide cache is empty. You may want to run 'apt-file update'
as root to update the cache. You can also run 'apt-file update' as
normal user to use a cache in the user's home directory.
Processing triggers for initramfs-tools (0.122ubuntu8.8) ...
update-initramfs: Generating /boot/initrd.img-4.13.0-41-generic

gzip: stdout: No space left on device
E: mkinitramfs failure find 141 cpio 141 gzip 1
update-initramfs: failed for /boot/initrd.img-4.13.0-41-generic with 1.
dpkg: error processing package initramfs-tools (--configure):
 subprocess installed post-installation script returned error exit status 1
No apport report written because MaxReports is reached already
                                                              Errors were encountered while processing:
 linux-firmware
 linux-image-4.13.0-43-generic
 linux-image-extra-4.13.0-43-generic
 linux-image-generic-hwe-16.04
 linux-generic-hwe-16.04
 linux-image-4.4.0-124-generic
 linux-image-4.4.0-127-generic
 linux-image-extra-4.13.0-41-generic
 linux-image-extra-4.4.0-124-generic
 linux-image-extra-4.4.0-127-generic
 linux-image-generic
 linux-image-extra-virtual
 initramfs-tools
E: Sub-process /usr/bin/dpkg returned an error code (1)

t$ sudo apt remove --purge linux-image-4.4.0-127-generic linux-image-extra-4.4.0-127-generic 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be REMOVED:
  linux-image-4.4.0-127-generic* linux-image-extra-4.4.0-127-generic* linux-image-extra-virtual* linux-image-generic*
0 upgraded, 0 newly installed, 4 to remove and 212 not upgraded.
4 not fully installed or removed.
After this operation, 224 MB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 260557 files and directories currently installed.)
Removing linux-image-extra-virtual (4.4.0.127.133) ...
Removing linux-image-generic (4.4.0.127.133) ...
Removing linux-image-extra-4.4.0-127-generic (4.4.0-127.153) ...
depmod: FATAL: could not load /boot/System.map-4.4.0-127-generic: No such file or directory
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
update-initramfs: Generating /boot/initrd.img-4.4.0-127-generic
run-parts: executing /etc/kernel/postinst.d/pm-utils 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
run-parts: executing /etc/kernel/postinst.d/unattended-upgrades 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
run-parts: executing /etc/kernel/postinst.d/update-notifier 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
run-parts: executing /etc/kernel/postinst.d/zz-update-grub 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-4.13.0-43-generic
Found initrd image: /boot/initrd.img-4.13.0-43-generic
Found linux image: /boot/vmlinuz-4.13.0-41-generic
Found initrd image: /boot/initrd.img-4.13.0-41-generic
Found memtest86+ image: /memtest86+.elf
Found memtest86+ image: /memtest86+.bin
done
Purging configuration files for linux-image-extra-4.4.0-127-generic (4.4.0-127.153) ...
Removing linux-image-4.4.0-127-generic (4.4.0-127.153) ...
Examining /etc/kernel/postrm.d .
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
update-initramfs: Deleting /boot/initrd.img-4.4.0-127-generic
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-4.13.0-43-generic
Found initrd image: /boot/initrd.img-4.13.0-43-generic
Found linux image: /boot/vmlinuz-4.13.0-41-generic
Found initrd image: /boot/initrd.img-4.13.0-41-generic
Found memtest86+ image: /memtest86+.elf
Found memtest86+ image: /memtest86+.bin
done
Purging configuration files for linux-image-4.4.0-127-generic (4.4.0-127.153) ...
Examining /etc/kernel/postrm.d .
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 4.4.0-127-generic /boot/vmlinuz-4.4.0-127-generic
```
