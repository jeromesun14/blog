title: Linux开发环境配置记录
date: 2015-09-10 21:13:24
toc: true
tags: [Linux, ubuntu, 远程登录, 文件共享, 串口]
categories: linux
keywords: [ubuntu, remmina, freerdp, serialport, kermit, samba, tftp, windows, 远程登陆, samba 密码错误, 拷贝粘贴, 失败, 串口, lrz, wine, 共享, ser2net, comfoolery, linux, 启机进入命令行, startup to command line]
description: Linux开发环境常用配置记录，含远程登录、文件共享、串口访问与共享、交叉编译器、终端等内容。
---

## 远程登录windows

### remmina
安装包：

* `remmina`
* `remmina-plugin-rdp`

配置：

1. Protocol，选择“RDP - Remote Desktop Protocol”
2. Basic标签
   * Server，windows的IP
   * User name
   * Password
3. Advanced标签
   * Security，选择“Negotiate”
4. SSH标签
   * 去掉Enable SSH tunnel

<!--more-->

### freerdp
本人倾向使用freerdp，简单易用。

安装包：`freerdp`。
命令：（将username / password / 1.1.1.1替换成自己的内容）
`xfreerdp -u username -p "password" -g 1024x768 -x l --plugin cliprdr --plugin rdpsnd 1.1.1.1`

因为版本不同，可能命令参数不一样：
`xfreerdp --sec rdp -u username -p "password" -g 1440x790 -x l --plugin cliprdr --plugin rdpsnd 1.1.1.1`

#### 跨系统拷贝粘贴失败问题
注意，windows 7有bug，导致clipboard有问题。即windows与Linux的copy、paste无法共用。解决方法见[链接](https://social.technet.microsoft.com/Forums/windows/zh-CN/5bbc11e8-ca2d-41ac-b640-d66ce971f58f/copy-paste-clipboard-issues-not-working)，其中mike 1504的做法，通过任务管理器杀掉rdpclip.exe再重启rdpclip.exe进程规避本问题。


>This is an old problem.  It has been around since pre2004, and microsoft is well aware of it.  The cause is tied to MS Office as well as the OS.  It usually occurs during Remote desktop sessions, and the problem is on the remote machine.  The best solution I have found is to pull up task man, end the process rdpclip.exe, and then restart it by first changing focus from task manager to another program and back to task manager, choose file>new task (run) and then typing in rdpclip.exe and hit enter.  The remote clip service will restart, and usually the copy you wanted to paste immediately becomes available.
>If you fail to change focus from Task Manager to another program and back you may find the remote session locks up.  If so, don't panic, simply end the remote session and begin a new one.
>
>In case you don't know how to access task manager on the remote machine (CTRL ALT DELETE is always to the local machine), simply right click on the task bar and choose it from the context menu.
>I would have thought by win7 this would have been fixed, but I am running Win7 enterprise trial right now and having the problem there.

### ssh远程登录Linux
安装包：`openssh-server`  
客户端登录：`ssh username@hostIP`

## 串口
### PAC Manager
安装包：

* PAC Manager，不在Ubuntu仓库中，Sourceforge[链接](http://sourceforge.net/projects/pacmanager/)。
* `cu`

需要把/dev/ttyUSB0的用户切换成cu的uucp才能被cu使用。
使用cu时需要用sudo
还没有明白remote-tty怎么用。

### kermit
安装包：

* `ckermit`
* `lrzsz`，用于xmodem传输

不改权限的话，只有用超级用户才能正常用kermit访问串口。
```s
sudo kermit
set line /dev/ttyUSB0
set speed 115200
set carrier-watch off
set handshake none
set flow-control none
robust 

set receive packet-length 9024
set send packekt-length 9024

set protocol xmodem
```

在ckermit命令提示符下：

* connect，连接串口；
* ?，查看所有命令，前面的配置命令在此都可以看到；
* send /path/to/file，传输文件，支持相对路径；
* 在串口控制台下，通过ctrl + \，再敲c，返回ckermit控制台。

### Ser2net

共享串口（SerialPort），提供telnet服务。

配置文件`/etc/ser2net.conf`，类似下面的配置依葫芦画瓢就可以了。

* 2001，端口号
* telnet，用telnet访问串口
* /dev/ttyUSB1，表示实际串口
* 9600，波特率
* 8DATABITS NONE 1STOPBIT，常见的8N1
* 600，port timeout，如果在超时时间间隔内没有对串口进行任何操作，ser2net会自动退出。

可以将所有的波特率的配置都添加进去，注意使用不同的端口号。

```
2001:telnet:600:/dev/ttyUSB1:9600 8DATABITS NONE 1STOPBIT -XONXOFF -RTSCTS banner 
3001:telnet:600:/dev/ttyUSB1:115200 8DATABITS NONE 1STOPBIT -XONXOFF -RTSCTS banner 
```

Windows下，可使用[`Comfoolery`](https://github.com/sunnogo/comfoolery)共享串口，功能更加强大。


### wine连接不上串口
来源[链接](http://goodgreyhound.com/usb-serial-port-converter-for-wine/)。
步骤：

* 映射/dev/ttyUSB0成windows中的概念com1

```
ln -s /dev/ttyUSB0 ~/.wine/dosdevices/com1
```

* 查看/dev/ttyUSB0的用户组，如下，用户组为dialout

```
ls -l /dev/ttyUSB0
crw-rw---T 1 root dialout 4, 6 Nov 18 20:34 ttyUSB0
```

* 将当前用户加入用户组dialout


```
sudo vi /etc/group

修改
dialout:x:20:
为
dialout:x:20:your_user_name
```

## 文件共享

### samba
安装包：`samba`

配置：

* 修改配置文件`/etc/samba/smb.conf`，确认要共享的目录。`sudo service samba restart`生效。配置样例如下。
* 创建samba用户：`sudo smbpasswd -a user_name`，user_name填成自己想要的名称。

```
[linuxMint]
    comment = linux
    path = /home/sunyongfeng/
    writeable = yes
    valid users = sunyongfeng
```
其中，linuxMint是共享目录显示给使用者的名称。path是实际共享的目录，valid users是合法的用户。

#### windows 7 如何清除自动登录的凭据

`控制面板` -> `用户账户` -> `管理您的凭据`，然后单击要清除的那个凭据，点击`从保管库中删除`，重启生效。

解决Ubuntu用户名变动后，无法从win7 登录 samba。

### Linux 访问 Windows 共享目录 

1. 在 windows 设置好共享目录；
2. 使用 `smbclient` 访问共享目录（可测试共享目录是否可用）： `smbclient //IP/share_dir -U your_username` ；
3. 挂载共享目录到 linux： `sudo mount -t cifs -o username=your_username,password=your_passwd //IP/share_dir /mnt`
4. 亦可直接在文件管理器中直接输入`smb://IP`，按提示输入用户名密码，即可从 Linux 文件管理器中直接查看 windows 的共享目录。

* `smbclient` 访问 log：

```
sunyongfeng@sunnogo:~$ smbclient //IP/share_dir -U your_name
Enter oa's password:
Domain=[sunnogo] OS=[Windows 7 Ultimate 7601 Service Pack 1] Server=[Windows 7 Ultimate 6.1]
smb: \> ls
  .                                   D        0  Thu Mar 10 19:24:05 2016
  ..                                  D        0  Thu Mar 10 19:24:05 2016
  asic                                D        0  Thu Mar 10 18:58:41 2016
  linux                               D        0  Thu Mar 10 18:58:43 2016
  python                              D        0  Thu Mar 10 18:59:04 2016
  工具                              D        0  Thu Mar 10 19:06:10 2016
  项目                              D        0  Thu Mar 10 20:08:25 2016

                40960 blocks of size 4194304. 33262 blocks available
smb: \>
```

* `mount`挂载 log：
```
sunyongfeng@sunnogo:~$ sudo mount -t cifs -o username=your_username,password=your_passwd //IP/share_dir /mnt
sunyongfeng@sunnogo:~$ ls -al /mnt/
总用量 3244
drwxr-xr-x 2 root root    4096  3月 10 19:24 .
drwxr-xr-x 3 root root    4096  3月 11 09:35 ..
drwxr-xr-x 2 root root       0  3月 10 18:58 linux
drwxr-xr-x 2 root root       0  3月 10 18:59 python
drwxr-xr-x 2 root root       0  3月 10 19:06 工具
drwxr-xr-x 2 root root       0  3月 10 20:08 项目
```

### tftp
安装包：`tftpd-hpa`

配置：

* 修改配置文件`/etc/default/tftpd-hpa`，样例如下。
* 需要`chmod 777 /home/sunyongfeng/tftpboot`，否则可能出现权限问题，`TFTP error: 'Permission denied' (0)`。tftpboot为tftp共享的主目录。

```
# /etc/default/tftpd-hpa  

TFTP_USERNAME="tftp"
#TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_DIRECTORY="/home/sunyongfeng/tftpboot"
TFTP_ADDRESS="[::]:69"
#TFTP_OPTIONS="--secure"                                                                            
TFTP_OPTIONS="-l -c -s"
```

**异常**：

* 如果一直timeout，可能是网卡出问题了，`sudo service networking restart`，或者重启电脑。
* 有时不清楚为何启机之后，tftpd-hpa在，但是tftp无法下载。`sudo service tftpd-hpa restart`可解决此问题。没有看系统log细究原因。

### atftpd
另一款tftp服务器。

安装包：`atftpd`
配置：

* 修改`/etc/default/atftpd`，样例如下。
* 把`/etc/inet.conf`中的tftp注释掉，不然会出现“atftpd: cannot bind port: 69/udp"。

**限制**：最大文件有限制，32M。


```
#/etc/default/atftpd

USE_INETD=false    
OPTIONS="--tftpd-timeout 300 --retry-timeout 5 --mcast-port 1758 --mcast-addr 239.239.239.0-255 --mcast-ttl 1 --maxthread 100 --verbose=7 /home/sunyongfeng/tftpboot"
```


## 安装linaro gcc交叉编译器
```
sudo add-apt-repository ppa:linaro-maintainers/toolchain
sudo apt-get update
sudo apt-get install gcc-arm-linux-gnueabi
```

## 终端（Terminal）

### guake

### fbterm

### screen

* screen -S sessionName，为 screen 会话取名字
* screen -list，查看当前有哪些会话
* screen -r sessionName，Reattach 到 sessionName 会话
* ctrl + a + d，detach 当前会话

### script

## 启机进入命令行
> `/etc/default/grub`, change this line
> `RUB_CMDLINE_LINUX_DEFAULT="splash quiet"` to
> `GRUB_CMDLINE_LINUX_DEFAULT="text"`
> Run `sudo update-grub` when done. This is easily reversed too if you need to change back.


