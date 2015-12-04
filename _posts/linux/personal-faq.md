title: Linux使用记录
date: 2015-09-02 20:03:22
toc: true
tags: [Linux, ubuntu, apt]
categories: linux
keywords: [ubuntu, apt, 网络配置, 中文支持, fcitx, grub]
description: Linux装机后的常用配置、软件说明，及使用问题记录。
---

推荐使用Server类iso安装系统，原因——干净！

善用/var/log
----------------
该目录存放系统日志，很多异常，比如进不了GUI、服务器连不上等问题，都可通过此目录内的log排查。

比如samba连不上，可通过`cat /var/log/auth.log | grep samba`查看samba失败的原因。

服务重启
------------
很多服务如samba、tftp如果配置文件改了之后，需要通过`sudo service smbd/tftp restart`来重启服务。

<!--more-->

硬件相关
-----------

### 网卡相关
以Ubuntu server 14.04为例。

#### 静态IP配置
配置文件：`/etc/network/interfaces`，如eth0的配置。

```
sunnogo@sunyongfeng:~/Downloads$ cat /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# eth0, Realtek Fast Ethernet NIC
auto eth0
iface eth0 inet static 
address 192.168.204.148
netmask 255.255.255.0
gateway 192.168.204.1
```

以上配置相当于命令：
```
ifconfig eth0 192.168.204.148 netmask 255.255.255.0
route add default gw 192.168.204.1
```

#### DNS配置
配置文件：`/etc/resolvconf/resolv.conf.d/base`

```
sunnogo@sunyongfeng:~/Downloads$ cat /etc/resolvconf/resolv.conf.d/base 
nameserver 192.168.58.110
nameserver 192.168.58.111
nameserver 8.8.8.8
```

#### 禁用IPv6
禁用的原因：ubuntu源默认使用IPv6，详见[链接](http://linux.cn/article-3373-1.html)。

修订文件`/etc/sysctl.conf`，在该文件最后加上：
```
# IPv6 disabled
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
保存关闭，并执行
```
$ sudo sysctl -p
```
即时生效，可通过ifconfig检查。重启有效。

### 显示相关

#### 屏幕分辨率问题
处理elementary OS时的操作

```
sudo xrandr --newmode "1440x900" 106.50 1440 1528 1672 1904 900 903 909 934 -hsync +vsync
sudo xrandr --addmode VGA1 1440x900
sudo xrandr --output VGA1 --mode 1440x900
```

日常使用
-----------
### apt

#### 配置源

* 安装ubuntu的图形工具`update-manager`，通过`Software & Updates`，在“Ubuntu Software”标签页，点“Download from:”旁边的复选框，选择`http://mirrors.yun-idc.com/ubuntu`，这个源在福州的速度不错。
* 亦可通过更改`/etc/apt/source.list`的内容实现源配置。
* debian可以通过`apt-spy`实现类似功能。

#### 更新cache
`apt-get update`

#### 以关键字查找包
`apt-cache search your-keyword`

#### 强制使用IPv4更新
默认使用IPv6源更新，不过国内目前还未全力推广IPv6。通过`Software & Updates`图形界面配置中文源后，无此问题。

强制使用IPv4更新的方法，说明详见[链接](http://unix.stackexchange.com/questions/9940/convince-apt-get-not-to-use-ipv6-method)。

* 使用`apt-get install`时，加入选项 `-o Acquire::ForceIPv4=true`。
* 添加一个配置文件`/etc/apt/apt.conf.d/99force-ipv4`，在配置文件中写入`Acquire::ForceIPv4 "true";`。

#### 安装失败
出现如下错误：

```
sunyongfeng@R04220 ~/smoking/ci $ sudo apt-get install npm
[sudo] password for sunyongfeng: 
	正在读取软件包列表... 有错误！
	E: Encountered a section with no Package: header
E: Problem with MergeList /var/lib/apt/lists/packages.linuxmint.com_dists_rebecca_import_i18n_Translation-en
	E: 无法解析或打开软件包的列表或是状态文件。
```

解决，简单粗暴地删除cache，再更新一下。
```
sudo rm /var/lib/apt/lists/* -vf
sudo apt-get update
```

### 中文支持

#### Ubuntu server 14.04中文支持

默认使用English，之后装gdm，想在英文环境中显示中文。

安装的包：

* 中文支持
   gnome中文支持包：
   + `language-pack-zh-han*`
   + `language-pack-gnome-zh-han*`
* 中文字体
   + `ttf-wqy-microhei`
   + `ttf-wqy-zenhei`
   + `ttf-ubuntu-font-family`，这个不是中文字体，只是装一下而已。
   + `fonts-arphic-uming`，个人很喜欢的一个中文字体，不过其英文显示有点难看。 
 
### 输入法

使用经典的`fcitx`。

#### Ubuntu server 14.04 fcitx
* 安装的包
	+ `fcitx`
	+ `fcitx-config-gtk2`，图形配置工具
	+ `fcitx-table-wbpy`，五笔拼音输出法
	+ `fcitx-module-*`，有`dbus`、`kimpanel`等
	+ `fcitx-frontend-*`，在gdm中，不需要装这个。为`fbterm`准备。
* 配置
	+ 卸载ibus，`apt-get remove ibus`，5M多；原理不明。
	+ 通过`fcitx-config-gtk2`，点击“Input Method”标签页左下方的“+”号，搜索出WubiPinyin，添加即可。

#### fcitx旧安装方式记录

```
sudo apt-get install fcitx fcitx-frontend-qt5 fcitx-table-wbpy fcitx-ui-classic fcitx-config-gtk3
sudo apt-get remove ibus
ps -e | grep ibus, kill掉带ibus_daemon的那条进程
sudo apt-get install im-switch
in-switch -s fcitx
```

fcitx-table-xxx，这里用的wbpy是五笔拼音，其他输入法如拼音由使用者自己选择。
执行：fcitx-autostart可开机自启动打开fcitx。

### 浏览器

安装的包：（安装完后，默认的中文字体意外地显示得很好`AR PL UMing CN`和`Monospace`）

* `chromium-browser`
* `pepperflashplugin-nonfree`

### 文件管理器（File Manager）
注意与窗口管理器（Window Manager）的概念不一样。

安装包：

* gnome，`nautilus`
* kde，`dolphin`

### 文本编辑器

安装包：
* gnome，`gdeit`
* kde，`kate`

### 引导失败
一般是先装Linux再装Windows出现。解决方法：

* 用`Docker`吧，不用再装Windows了。
* 通过Linux USB启动盘进入U盘中的Linux，通过grub一般都能修复，然后替换掉windows的引导，先通过grub引导。
	* `update-grub`或`update-grub2`
	* `grub-install`
* 通过`EasyBCD`配置，重启后进入Linux，然后再进行grub修复。

### 键鼠

#### 提示CapsLock键
大写提示工具：`indicators-keylock`，先导入源，再安装。

```
sudo add-apt-repository ppa:tsbarnes/indicator-keylock && sudo apt-get update
sudo apt-get install indicator-keylock
```

#### 禁用触摸板
Ubuntu配置禁用触摸板[wiki](http://wiki.ubuntu.org.cn/%E8%A7%A6%E6%91%B8%E6%9D%BF)。

命令：`synclient touchpadoff=1`。

可以使用工具`GPointingDeviceSettings`。

### 用户管理

#### 添加用户
命令：

* `useradd`
* `usermod`
* `userdel`

#### 修改密码
命令：`passwd username`

##### 修改root密码
ubuntu的默认root密码随机，通过`sudo passwd root`修改

#### 添加sudo用户
如果用户不是sudo用户，且使用sudo命令，则会提示“xxx is not in the sudoers file. This incident will be reported”。解决方法：

* 拿到root权限，添加文件`/etc/sudoers`读权限
* 修改`/etc/sudoers`，在`User privilege specification`下，类似root那样，添加一行`usernamexxxx	ALL=(ALL:ALL) ALL`

```
# User privilege specification
root	ALL=(ALL:ALL) ALL
usernamexxxx	ALL=(ALL:ALL) ALL
```

