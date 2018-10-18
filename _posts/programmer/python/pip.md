title: pip 使用记录
date: 2018-10-18 15:52:00
toc: false
tags: [python, pip]
categories: python
keywords: [python, pip, 国内源, docker, Dockerfile, fakeroot, 编译, proxy]
description: pip 使用记录。
---

## 替换源

* 全局替换

参考 http://mirrors.ustc.edu.cn/help/pypi.html, Unix 环境: `$HOME/.config/pip/pip.conf`:

```
[global]
index-url = https://mirrors.ustc.edu.cn/pypi/web/simple
format = columns
```

如果源不支持 https，需要再加一行：`trusted-host = mirrors.ustc.edu.cn`

> 使用 pip 时如果出现 configparser.MissingSectionHeaderError: File contains no section headers., 说明你的 pip.conf 忘记加上 [global] 这一行了。

* pip install 带参数替换

```
PIP_OPTIONS="--index-url=http://xxx.com/pypi/web/simple/ --trusted-host=xxx.com"
sudo LANG=C chroot $FILESYSTEM_ROOT pip install $PIP_OPTIONS 'scapy'
```

* Dockerfile 中替换
在 Dockerfile 中加一条命令：

```
RUN mkdir -p ~/.pip && echo "[global]\nindex-url = https://mirrors.ustc.edu.cn/pypi/web/simple/\ntrusted-host = mirrors.ustc.edu.cn" > ~/.pip/pip.conf
```

## 安装本地 whl 包

需要带**全路径**安装：`pip install /full/path/to/xxx.whl`
