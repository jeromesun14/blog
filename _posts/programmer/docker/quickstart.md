title: docker 快速入门
date: 2017-04-19 13:54:20
toc: true
tags: [Linux, ubuntu, docker]
categories: docker
keywords: [ubuntu, docker, copy, cp, container]
description: 如何从 host 拷贝文件到 docker 容器中。
---

## 安装 docker

一键安装 [docker](https://docs.docker.com/engine/installation/)：`wget -qO- https://get.docker.com/ | sh`。

## 基本操作

1. 运行容器，docker run 有多种选项。`docker run -d --net=host --privileged -t -v workshop:/workshop --name jerome debian:jessie bash`
   + `-v` 选项，将 host 目录挂载到 docker 容器系统中
   + `--name your_name`，为运行的容器命名，相信直接操作名字总比操作一串数字方便、可用
2. 查看运行中的容器，`docker ps`
3. 容器退出后，还会缓存在系统中，
   + 查看缓存，`docker ps -a`
   + 运行缓存，`docker start jerome`，
   + 删除缓存，`docker rm jerome`
4. 进入容器 shell 命令行，`docker exec -it jerome bash`，不会像 `docker attach` 进去退出后，直接将在跑的 docker 实例退出来。
5. 怕误操作导致缓存丢失？那就提交一下，`docker commit jerome debian:jessie_jerome`
6. 查看本地镜像，`docker images`
7. 重命名镜像，`docker tag image-id REPOSITORY[:TAG]`
8. 复制文件到容器中，`docker cp abc jerome:/` or `docker cp jerome:/abc host_dir/abc`

进入正在运行的容器命令行样例：

```
sunyongfeng@openswitch-OptiPlex-380:~$ docker ps
CONTAINER ID        IMAGE                 COMMAND             CREATED              STATUS              PORTS               NAMES
dab2267a06dd        p4dockerswitch_bmv2   "/bin/bash"         About a minute ago   Up About a minute                       elastic_morse
sunyongfeng@openswitch-OptiPlex-380:~$ docker exec -i -t dab2267a06dd /bin/bash
root@dab2267a06dd:/# 
root@dab2267a06dd:/# ls
bin   dev  home  lib64  mnt            nanomsg-1.0.0.tar.gz  opt        proc  run   srv  third-party   thrift-0.9.2.tar.gz  usr
boot  etc  lib   media  nanomsg-1.0.0  nnpy                  p4factory  root  sbin  sys  thrift-0.9.2  tmp                  var
root@dab2267a06dd:/#
```

## docker 使用中国源
见 https://www.docker-cn.com/registry-mirror

* 方法1：

```
$ docker pull registry.docker-cn.com/library/ubuntu:16.04
```

* 方法2：拉取的时候加参数 `--registry-mirror=https://registry.docker-cn.com `

* 方法3：默认配置，需要重启 docker 服务才能生效

为了永久性保留更改，您可以修改 /etc/docker/daemon.json 文件并添加上 registry-mirrors 键值。

```
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

修改保存后重启 Docker 以使配置生效。


## 本地 repo
详见 https://docs.docker.com/registry/，参考 https://www.jianshu.com/p/fc544e27b507。

## docker 命令没有权限
必须 sudo docker xxx？

见 [使用docker第一步](http://wiki.jikexueyuan.com/project/docker/articles/basics.html):

本指南假设你已经完成了Docker的安装工作。检查你安装的Docker,运行以下命令：

```
# Check that you have a working install
$ docker info
```

如果你得到 `docker: command not found`，你可能没有完整的安装上Docker。
如果你得到 `/var/lib/docker/repositories: permission denied`，那你可能没有权限访问你主机上的Docker。

为获得访问Docker权限可以直接在命令前加sudo，或者采取以下步骤授予权限：：

```
# 如果还没有docker group就添加一个：
$ sudo groupadd docker
# 将用户加入该group内。然后退出并重新登录即可生效。
$ sudo gpasswd -a ${USER} docker
# 重启docker
$ sudo service docker restart
```

## docker 空间不足
配置 docker 的 basesize。

```
sudo systemctl stop docker
/etc/docker/daemon.json
{
  "storage-driver": "devicemapper"
}
sudo dockerd --storage-opt dm.basesize=50G
sudo systemctl start docker
```

## 修改 docker storage driver 为 aufs

ubuntu 16.04 默认为 overlay2，但是目前在用的 sonic 不支持，因此只能回退为 aufs。可通过 `docker info` 命令查看，其中有一句为：

```
Storage Driver: overlay2
```

ubuntu 16.04 如何支持 aufs？详见 [Use the AUFS storage driver](https://docs.docker.com/engine/userguide/storagedriver/aufs-driver/)。


0. 查看本机 docker storage driver 是否为 aufs，`docker info`，
1. 备份 docker 数据，`sudo cp /var/lib/docker /var/lib/docker.bak`
2. 停止 docker 服务，`sudo service docker stop`
3. 确认系统是否支持 aufs，`grep aufs /proc/filesystem`
4. 如果系统不支持 aufs，安装依赖包，`sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual`
5. 加载 aufs 内核模块，`sudo modprobe aufs`
6. 配置 docker 启动参数，依赖 `/etc/docker/daemon.json` 为以下内容。
6. 启动 docker 服务，`sudo service docker start`
7. 确认 docker storage driver 已改为 aufs，`docker info`

/etc/docker/daemon.json 内容：

```
{                                                                                                   
    "storage-driver": "aufs"                                                                        
} 
```
