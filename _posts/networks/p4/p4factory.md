title: p4factory使用记录
date: 2017-04-17 17:34:20
toc: true
tags: [p4, int]
categories: p4
keywords: [p4, factory, int, inband telemetry]
description: P4 factory 样例使用记录
---


# 问题记录

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4factory$ git submodule update --init --recursive
Cloning into 'submodules/infra'...
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
fatal: clone of 'git@github.com:floodlight/infra' into submodule path 'submodules/infra' failed
Failed to recurse into submodule path 'submodules/p4ofagent/submodules/indigo/submodules/bigcode'
Failed to recurse into submodule path 'submodules/p4ofagent/submodules/indigo'
Failed to recurse into submodule path 'submodules/p4ofagent'
```


## 安装 ptf

`python setup --install --prefix=$P4HOME/install`

```
You are attempting to install a package to a directory that is not
on PYTHONPATH and which Python does not read ".pth" files from.  The
installation directory you specified (via --install-dir, --prefix, or
the distutils default setting) was:

    /usr/local/lib/python2.7/site-packages/

and your PYTHONPATH environment variable currently contains:

    ''

Here are some of your options for correcting the problem:

* You can choose a different installation directory, i.e., one that is
  on PYTHONPATH or supports .pth files

* You can add the installation directory to the PYTHONPATH environment
  variable.  (It must then also be on PYTHONPATH whenever you run
  Python and want to use the package(s) you are installing.)

* You can set up the installation directory to support ".pth" files by
  using one of the approaches described here:

  http://packages.python.org/distribute/easy_install.html#custom-installation-locations

Please make the appropriate changes for your system and try again.
```

解决：export 

## switch configure 失败

pdfixed 头文件找不到

bmv2 configure 时要加上 pdfixed

