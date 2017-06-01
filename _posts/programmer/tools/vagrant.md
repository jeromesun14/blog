title: vagrant 使用记录
date: 2017-03-31 11:39:24
toc: true
tags: [vagrant, virtualBox]
categories: programmer
keywords: [vagrant, virtualBox, ping, failed, NAT, 网络配置, network configuration, private network, bridge, public network, vagrantfile, 端口映射, 失败, failed, ubuntu, 16.04, 默认密码, default password, 下载位置, download location, 打包, package, 集群, 多机, cluster, 外网访问, 服务, service, mapping]
description: Vagrant + virtualBox 使用记录
---

# vagrant 如何 ping 通虚拟机

## vagrant 默认网络通信机制
vagrant 默认使用 NAT 端口映射，可以通过端口映射访问虚机内部端口号。但是默认情况下，vagrant 可以访问外网，但是外部世界访问不了 vagrant （类似以前在学校，学校内可以访问外网，但是外网访问不了教育网）。
如果你想在外部访问 vagrant 虚机提供的服务，需要配置端口映射，比如将虚机的端口 80 映射成 host 机的端口 8080，详见 [vagrant networking basic usage](https://www.vagrantup.com/docs/networking/basic_usage.html)：

```vagrantfile
  config.vm.network "forwarded_port", guest: 80, host: 8080
```

默认情况下，vagrant 虚机中有一张网卡，默认IP `10.0.2.15`。虚机可上外网，host 无法 ping 通虚机。

## vagrant 其他通信机制

### private network
详见[vagrant networking private_network](https://www.vagrantup.com/docs/networking/private_network.html)。

通常利用 provider 创建的虚拟网卡，例如 virtualBox 的 Host-Only Network。

```
以太网适配器 VirtualBox Host-Only Network:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::a801:a2e7:cbd9:7126%3
   IPv4 地址 . . . . . . . . . . . . : 192.168.56.1
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . :
```

vagrant 的配置：

* 静态IP：

```
  config.vm.network "private_network", ip: "192.168.56.10"
```

* 动态IP：

```
  config.vm.network "private_network", type: "dhcp"
```

配置后虚机中新增一张网卡，如下的 enp0s8。

```
vagrant@ubuntu-xenial:~$ sudo ifconfig -a
enp0s3    Link encap:Ethernet  HWaddr 02:6a:71:da:1d:13
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::6a:71ff:feda:1d13/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:45731 errors:0 dropped:0 overruns:0 frame:0
          TX packets:20045 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:37231206 (37.2 MB)  TX bytes:1387674 (1.3 MB)

enp0s8    Link encap:Ethernet  HWaddr 08:00:27:c4:aa:2c
          inet addr:192.168.56.10  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fec4:aa2c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:818 errors:0 dropped:0 overruns:0 frame:0
          TX packets:575 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:96862 (96.8 KB)  TX bytes:100624 (100.6 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:480 (480.0 B)  TX bytes:480 (480.0 B)
```

此是 host 可以 ping 通虚机。

```
C:\Users\R0>ping 192.168.56.10

正在 Ping 192.168.56.10 具有 32 字节的数据:
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.56.10 的回复: 字节=32 时间<1ms TTL=64

192.168.56.10 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 0ms，最长 = 0ms，平均 = 0ms

C:\Users\R0>
```

### public network
利用 provider 提供的 bridge （桥接功能）。

# vagrant halt 无法关机
失败 log：

```
E:\vagrant>Vagrant halt
==> default: Attempting graceful shutdown of VM...
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

shutdown -h now

Stdout from the command:



Stderr from the command:


E:\vagrant>
```

解决方法，见 [issue 1659](https://github.com/mitchellh/vagrant/issues/1659) @rfay 的回答。即将默认用户配置为 sudoers 后，还要在 /etc/sudoers 中添加 `vagrant ALL=(ALL) NOPASSWD:ALL`。

```
@dengzhp I've done this to myself any number of times by messing up the /etc/sudoers or /etc/sudoers.d. I did it today, which is why I landed here. Somewhere in /etc/sudoers (or /etc/sudoers.d if it's included) you have to have

vagrant ALL=(ALL) NOPASSWD:ALL
Defaults:vagrant !requiretty
without that, the vagrant ssh (without tty) fails mysteriously. I once again had built a machine without my puppet vagrant module, which adds this in.

@mitchellh if there's not already an FAQ on this, it's a good topic for one. I seem to do it over and over again :-)
```

不过我的环境中，/etc/sudoers 默认权限为只读，因此，直接将 `vagrant ALL=(ALL) NOPASSWD:ALL` 添加在 /etc/sudoers.d/username 中。

```
vagrant@ubuntu-xenial:~$ ls -al /etc/sudoers
-r--r----- 1 root root 755 Jan 20 16:01 /etc/sudoers
vagrant@ubuntu-xenial:~$ sudo cat /etc/sudoers.d/vagrant
vagrant ALL=(ALL) NOPASSWD:ALL
vagrant@ubuntu-xenial:~$ 
```

# vagrantfile 添加端口映射后 vagrant up 失败

添加一个新的端口映射到 Vagrantfile，`config.vm.network "forwarded_port", guest: 16385, host: 26385`。
之后执行 `vagrant up` 失败，log：

```
E:\vagrant>Vagrant reload
==> default: Attempting graceful shutdown of VM...
==> default: Clearing any previously set forwarded ports...
C:/HashiCorp/Vagrant/embedded/gems/gems/vagrant-1.9.3/lib/vagrant/util/is_port_open.rb:21:in `initialize': The requested address is not valid in its context. - connect(2) for "0.0.0.0" port 16385 (Errno::EADDRNOTAVAIL)
```

解决方法详见 [issue 8395](https://github.com/mitchellh/vagrant/issues/8395)，配置中加上 host_id，最终端口映射的配置为 `config.vm.network "forwarded_port", guest: 16385, host: 26385, host_ip: "127.0.0.1"`
```
Had the same problem on Windows 7.
This problem seems to be caused by the new host_ip parameter for the port forwarding feature.
I suggest that for compatibility reason, the default host_ip parameter should be set to 127.0.0.1 instead of 0.0.0.0
I managed to make the 1.9.3 version working by rewritten all my Vagrantfile(s) and adding the host_id: "127.0.0.1" parameter for each of the "forwarded_port" network configuration.
E.g.:
config.vm.network "forwarded_port", guest: 22, host: 1022, host_ip: "127.0.0.1", id: 'ssh'
```

# ubuntu 16.04 32-bit box 用户名密码反人类
一般情况下，vagrant box 用户名密码都为 vagrant，但是使用的第一个 box 却违反这个规则。[xenial32](https://atlas.hashicorp.com/ubuntu/boxes/xenial32) 的默认用户名为 `ubuntu`，密码为`7ea1ebda4a3d566ade4dd808`。

具体的 Box 内部 Vagrantfile 内容为：

```
# Front load the includes
include_vagrantfile = File.expand_path("../include/_Vagrantfile", __FILE__)
load include_vagrantfile if File.exist?(include_vagrantfile)

Vagrant.configure("2") do |config|
  config.vm.base_mac = "026A71DA1D13"
  config.ssh.username = "ubuntu"
  config.ssh.password = "7ea1ebda4a3d566ade4dd808"

  config.vm.provider "virtualbox" do |vb|
     vb.customize [ "modifyvm", :id, "--uart1", "0x3F8", "4" ]
     vb.customize [ "modifyvm", :id, "--uartmode1", "file", File.join(Dir.pwd, "ubuntu-xenial-16.04-cloudimg-console.log") ]
  end
end
```

# vagrant box add base ubuntu/xenial32  具体的下载位置

windows 下具体下载位置为 `C:\Users\yourName\.vagrant.d\boxes\base\`。

# vagrant 如何打包？
vagrant 打包事实上使用 tar + gzip 进行打包，后缀命名为 `.box`。
`vagrant package` 命令的打包结果据说不好使，没有实测。

# 外网如何使用 vagrant 服务
端口映射时，ip 填 `host IP`，而不是 `127.0.0.1` 本地地址。

如下配置，实测可在 172.x.x.x 网段 ssh 远程登陆虚机。

```
  config.vm.network "forwarded_port", guest: 16385, host: 26385, host_ip: "192.168.1.2"
  config.vm.network "forwarded_port", guest: 22, host: 2222, host_ip: "192.168.1.2"
```

ssh 远程登陆 log：

```
root@ubuntu:~# ssh vagrant@192.168.1.2 -p 2222
vagrant@192.168.1.2's password: 
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-115-generic i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Thu Apr  6 09:03:15 UTC 2017

  System load:  0.26              Processes:           78
  Usage of /:   4.2% of 39.34GB   Users logged in:     0
  Memory usage: 16%               IP address for eth0: 10.0.2.15
  Swap usage:   0%                IP address for eth1: 192.168.56.10

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

New release '16.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Thu Apr  6 09:03:16 2017 from 10.0.2.2
vagrant@vagrant-ubuntu-trusty-32:~$ 
```

未决问题：bridge 模式下，跨网段 ping，内网可以 ping 通外网，外网 ping 不通内网。

# 三机集群配置

支持变量，以程序的方式做配置。

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    (1..3).each do |i|
        config.vm.define "node#{i}" do |node|
            # 设置虚拟机的Box
            node.vm.box = "ubuntu/trusty32"
            # 设置虚拟机的主机名
            node.vm.hostname="node#{i}"
            # 设置虚拟机的IP
            node.vm.network "private_network", ip: "192.168.56.1#{i}"
            # 设置端口映射
            node.vm.network "forwarded_port", guest: 16385, host: "#{i}6385", host_ip: "127.0.0.1"
            node.vm.network "forwarded_port", guest: 22, host: "#{i}0022", host_ip: "127.0.0.1"
            
            node.vm.box_check_update = false
            
            # 设置主机与虚拟机的共享目录
            # node.vm.synced_folder "~/Desktop/share", "/home/vagrant/share"
            # VirtaulBox相关配置
            node.vm.provider "virtualbox" do |v|
                # 设置虚拟机的名称
                v.name = "node#{i}"
                # 设置虚拟机的内存大小  
                v.memory = 512
                # 设置虚拟机的CPU个数
                v.cpus = 1
            end
          
            # 使用shell脚本进行软件安装和配置
            # node.vm.provision "shell", inline: <<-SHELL
            #    
            # SHELL
            # end
        end 
    end
end
```
