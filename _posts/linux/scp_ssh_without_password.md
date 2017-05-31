title: 如何让 scp / ssh 不需要密码
date: 2017-04-25 13:58:24
toc: false
tags: [Linux, ubuntu, 远程登录, scp, ssh]
categories: linux
keywords: [ubuntu, scp, ssh, rsa, without, password, script]
description: 让 scp / ssh 不需要密码，可在脚本中直接使用。
---

> [How to use the Linux ‘scp’ command without a password to make remote backups](http://alvinalexander.com/linux-unix/how-use-scp-without-password-backups-copy)
> How to create a public and private key pair to use ssh and scp without using a password, which lets you automate a remote server backup process.

目标：通过 server 上的脚本，通过 cron 每天备份文件到 client。

* h1，sunyongfeng@192.168.204.168，client，用于保存服务器备份文件
* h2，root@172.18.111.192，server，需要通过脚本，每天备份文件到 192.168.204.168

步骤：

* 在 server 端生成 rsa 密钥对，`ssh-keygen -t rsa`

```
root@ubuntu:~# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c7:42:12:d7:40:58:16:0f:2b:29:6c:2b:67:6b:1c:e3 root@ubuntu
The key's randomart image is:
+--[ RSA 2048]----+
|      .+O+       |
|   .  .= +.      |
|    + + o .      |
|   . o + .       |
|  . *   S o      |
|   * +   o       |
|    E            |
|   .             |
|                 |
+-----------------+
root@ubuntu:~# 
root@ubuntu:~# ls .ssh/ -al            
total 20
drwx------ 2 root root 4096 Apr 25 14:05 .
drwx------ 7 root root 4096 Apr 25 13:51 ..
-rw------- 1 root root 1679 Apr 25 13:54 id_rsa
-rw-r--r-- 1 root root  393 Apr 25 13:54 id_rsa.pub
-rw-r--r-- 1 root root  444 Apr  6 16:58 known_hosts
```

* 拷贝公钥到 client 192.168.204.168，注意此时从 172.18.111.192 scp 拷贝时还需要密码。`scp .ssh/id_rsa.pub sunyongfeng@192.168.204.168:/home/sunyongfeng/.ssh/authorized_keys`

```
root@ubuntu:~# scp .ssh/id_rsa.pub sunyongfeng@192.168.204.168:/home/sunyongfeng/.ssh/authorized_keys
sunyongfeng@192.168.204.168's password: 
id_rsa.pub                                                                                       100%  393     0.4KB/s   00:00    
root@ubuntu:~# 
```

* 验证从 server 172.18.111.192 scp 到 192.168.204.168 已不需要密码。下文同时确认 ssh 远程 192.168.204.168 不需要密码。

```
root@ubuntu:~# scp .ssh/id_rsa.pub sunyongfeng@192.168.204.168:/home/sunyongfeng/.ssh/authorized_keys
id_rsa.pub                                                                                       100%  393     0.4KB/s   00:00    
root@ubuntu:~# 
root@ubuntu:~# ssh sunyongfeng@192.168.204.168
Welcome to Ubuntu 16.04 LTS (GNU/Linux 3.19.8-031908-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

234 packages can be updated.
0 updates are security updates.

Last login: Tue Apr 25 12:36:14 2017 from 192.168.204.167
sunyongfeng@openswitch-OptiPlex-380:~$ exit
logout
Connection to 192.168.204.168 closed.
root@ubuntu:~# 
```
