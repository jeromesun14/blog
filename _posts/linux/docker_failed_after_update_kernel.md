title: Linux升级内核后 docker 不可用
date: 2017-04-18 17:27:20
toc: true
tags: [Linux, ubuntu, docker]
categories: linux
keywords: [ubuntu, update, kernel, 更新, 内核, docker, 不可用, 16.04, 3.19, 4.4.0, fail, aufs, failure]
description: ubuntu 升级/降级 内核之后，docker 不可用问题及解决。
---

## 问题描述

因开发需求，降级 ubuntu 16.04 内核版本 4.4.0 为 3.19，之后 docker 启机失败。


## 问题 log

通过 `sudo systemctl status docker.service` 查看 docker 启动信息发现，docker 因 aufs 驱动问题无法正常启动。

log:

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ sudo service docker start
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ sudo systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since 二 2017-04-18 16:38:26 CST; 9s ago
     Docs: https://docs.docker.com
  Process: 3058 ExecStart=/usr/bin/dockerd -H fd:// (code=exited, status=1/FAILURE)
 Main PID: 3058 (code=exited, status=1/FAILURE)

4月 18 16:38:25 openswitch-OptiPlex-380 systemd[1]: Starting Docker Application Container Engine...
4月 18 16:38:25 openswitch-OptiPlex-380 dockerd[3058]: time="2017-04-18T16:38:25.914144290+08:00" level=info msg="libcontainerd: new containerd process, pid: 3064"
4月 18 16:38:26 openswitch-OptiPlex-380 dockerd[3058]: time="2017-04-18T16:38:26.962727971+08:00" level=error msg="[graphdriver] prior storage driver \"aufs\" failed:
4月 18 16:38:26 openswitch-OptiPlex-380 dockerd[3058]: time="2017-04-18T16:38:26.962806596+08:00" level=fatal msg="Error starting daemon: error initializing graphdriv
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: Failed to start Docker Application Container Engine.
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: docker.service: Unit entered failed state.
4月 18 16:38:26 openswitch-OptiPlex-380 systemd[1]: docker.service: Failed with result 'exit-code'.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ sudo systemctl start docker
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory/makefiles$ docker --version
Docker version 1.12.3, build 6b644ec
```

## 问题解决

见 [prior storage driver “aufs” failed: driver not supported Error starting daemon: error initializing graphdriver: driver not supported](http://stackoverflow.com/questions/33357824/prior-storage-driver-aufs-failed-driver-not-supported-error-starting-daemon) 中 Sergey Podobry 的答复：

> Try removing all downloaded images
> 
> `sudo rm -rf /var/lib/docker/aufs`
> 
> That helped me to recover docker after a kernel update.
> 
> Related issues on the github:
> 
> * https://github.com/docker/docker/issues/14026
> * https://github.com/docker/docker/issues/15651

本文通过备份的形式，重启 PC 后，docker 可正常运行。

```
sunyongfeng@openswitch-OptiPlex-380:/var/lib$ sudo mv /var/lib/docker /var/lib/docker.old        
[sudo] password for sunyongfeng: 
sunyongfeng@openswitch-OptiPlex-380:/var/lib$ sudo shutdown -r 0
...

sunyongfeng@openswitch-OptiPlex-380:~$ ps -e | grep docker
  717 ?        00:00:00 dockerd
  796 ?        00:00:00 docker-containe
sunyongfeng@openswitch-OptiPlex-380:~$
```
