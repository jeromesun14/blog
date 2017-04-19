title: windows 通过 vnc 远程 ubuntu
date: 2017-04-19 19:57:00
toc: true
tags: [ubuntu, 远程登录, vnc]
categories: linux
keywords: [ubuntu, linux, vnc, vnc4server, unity, gnome, chrome, 16.04]
description: windows 通过 vnc 远程 ubuntu 记录。
---

* ubuntu
  + 安装 vnc4server，`sudo apt-get install vnc4server`
  + 运行 vnc4server，配置密码，密码至少6个字符，至多8个字符
* windows
  + chrome 安装 vnc viewer 插件
  + 通过 ip:5901 远程登陆，默认使用 x-terminal-emulator & x-window-manager 界面。
* 配置使用 gnome 界面，[链接](https://askubuntu.com/questions/518041/unity-doesnt-work-on-vnc-server-under-14-04-lts)
  + 安装包，`sudo apt-get install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal`
  + 修改配置文件 `~/.vnc/xstartup`

```
#!/bin/sh

export XKL_XMODMAP_DISABLE=1
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
gnome-terminal &
```

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$ vncserver 

You will require a password to access your desktops.

Password:
Password must be at least 6 characters - try again
Password:
Verify:
Password too long - only the first 8 characters will be used

New 'openswitch-OptiPlex-380:1 (sunyongfeng)' desktop is openswitch-OptiPlex-380:1

Creating default startup script /home/sunyongfeng/.vnc/xstartup
Starting applications specified in /home/sunyongfeng/.vnc/xstartup
Log file is /home/sunyongfeng/.vnc/openswitch-OptiPlex-380:1.log

sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/mininet$
```
