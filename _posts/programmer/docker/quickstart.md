title: 拷贝文件到 docker 容器
date: 2017-04-19 13:54:20
toc: true
tags: [Linux, ubuntu, docker]
categories: docker
keywords: [ubuntu, docker, copy, cp, container]
description: 如何从 host 拷贝文件到 docker 容器中。
---

http://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container

重命名：docker tag image-id REPOSITORY[:TAG]

docker tag 20b119d0ceea p4dockerswitch_bmv2

docker ps

docker ps -a

docker run -it xxx /bin/bash

类似 ssh 进入 docker，直接起一个 bash 即可，docker exec -i -t $DID bash

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
