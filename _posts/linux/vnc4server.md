title: windows 通过 vnc4server 远程 ubuntu
date: 2017-04-19 19:57:00
toc: true
tags: [ubuntu, 远程登录, vnc]
categories: linux
keywords: [ubuntu, linux, vnc, vnc4server, unity, gnome, chrome, 16.04]
description: windows 通过 vnc4server 远程 ubuntu 记录。
---

## 程序安装与配置运行

linode 有一篇详尽的文章：[Install VNC on Ubuntu 16.04](https://www.linode.com/docs/applications/remote-desktop/install-vnc-on-ubuntu-16-04)。

* ubuntu
  + 安装 vnc4server，`sudo apt-get install vnc4server`
  + 运行 `vncpasswd`，配置密码，密码至少6个字符，至多8个字符
  + 运行 `vnc4server :1`，启动 vnc server。可通过 `vnc4server -kill :1` 关闭 vnc server。
* windows
  + chrome 安装 vnc viewer 插件
  + 通过 ip:5901 远程登陆，默认使用 x-terminal-emulator & x-window-manager 界面。
* 配置使用 gnome 界面，[参考链接](https://askubuntu.com/questions/518041/unity-doesnt-work-on-vnc-server-under-14-04-lts)
  + 安装包，`sudo apt-get install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal`
  + 修改配置文件 `~/.vnc/xstartup`

## log
### vnc4server 配置与运行

```
sunyongfeng@openswitch-OptiPlex-380:~$ vncpasswd 
Password:
Verify:
sunyongfeng@openswitch-OptiPlex-380:~$ 
sunyongfeng@openswitch-OptiPlex-380:~$ vncserver :1

New 'openswitch-OptiPlex-380:1 (sunyongfeng)' desktop is openswitch-OptiPlex-380:1

Starting applications specified in /home/sunyongfeng/.vnc/xstartup
Log file is /home/sunyongfeng/.vnc/openswitch-OptiPlex-380:1.log

sunyongfeng@openswitch-OptiPlex-380:~$ 
sunyongfeng@openswitch-OptiPlex-380:~$ vncserver -kill :1
Killing Xvnc4 process ID 2788
```

### 配置支持 gnome 界面

```
sunyongfeng@openswitch-OptiPlex-380:~$ cat .vnc/xstartup 
#!/bin/sh

# Uncomment the following two lines for normal desktop:
# unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
#x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#x-window-manager &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
gnome-terminal &
sunyongfeng@openswitch-OptiPlex-380:~$ 
```

## Screenshot
### chrome 插件

* 插件

![chrome-plugin VNC viewer](/images/linux/vnc/chrome-plugin-vncViewer.png)

* 输入 IP & 端口号

![access](/images/linux/vnc/vncViewer_access.png)

* 输入密码

![auth](/images/linux/vnc/vncViewer_access_authentication.png)

* xwindow 界面

![xwindow](/images/linux/vnc/vnc4server_xwindow.png)

* gnome 界面

![gnome](/images/linux/vnc/vnc4server_gnome.png)
