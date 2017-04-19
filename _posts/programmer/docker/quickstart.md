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
