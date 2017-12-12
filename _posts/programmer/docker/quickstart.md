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

## 镜像相关

* 查看本地镜像，`docker images`
* 重命名镜像，`docker tag image-id REPOSITORY[:TAG]`

## 运行容器

* 运行容器，`docker run -it xxx /bin/bash`，此为运行带 bash shell 的容器，默认进入命令行。
* 查看正在运行的容器，`docker ps [-a]`
* 类似 ssh 访问 docker 容器（进入命令行进行操作）。`docker exec -i -t $DID bash`，不会像 docker attach 进去退出后，直接将在跑的 docker 实例退出来。
* 拷贝文件到 docker 容器。`docker cp` 命令，参考 [stackoverflow](http://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container)。

```
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

连接容器命令行样例：

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
