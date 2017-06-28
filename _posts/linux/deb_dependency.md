title: 包管理工具依赖关系管理
date: 2017-06-28 10:42:00
toc: true
tags: [ubuntu, deb]
categories: linux
keywords: [ubuntu, linux, deb, package, control, dependency, dependencies, rpm, build dependency, pip, npm]
description: 研究包管理工具的依赖关系管理。
---

因产品的无法按 app 升级，发现产品中的组件已按包管理工具打包，但是却没有用上包管理工具的核心——依赖关系管理。本文研究包管理工具的依赖关系管理。

通常依赖关系管理分为安装时/运行时依赖和构建时依赖。这里主要讲安装时/运行时依赖。我所理解的安装时依赖即运行时依赖。
常见的包管理工具有 deb / rpm / pip / npm / rake 等，这里以 deb 为例。

详见 [How to make a "Basic" .deb](https://ubuntuforums.org/showthread.php?t=910717)。为验证安装时依赖关系管理，创建 mydep_2.24-1。helloworld_1.0-1 依赖 mydep_2.24-1。

## 目录架构

* helloworld 软件包目录架构

```
sunyongfeng@ubuntu:~/workshop/test$ tree helloworld_1.0-1
helloworld_1.0-1
├── DEBIAN
│   └── control
└── usr
    └── local
        └── bin
            └── helloworld
```

* mydep 软件包目录架构

```
4 directories, 2 files
sunyongfeng@ubuntu:~/workshop/test$ tree mydep_2.24-1
mydep_2.24-1
├── DEBIAN
│   └── control
└── usr
    └── local
        └── bin
            └── mydep

4 directories, 2 files
```

## 控制文件

该实验主要用到两个字段：

1. Version，本 package 版本号
2. Depends，本 package 依赖哪些软件包，及依赖的版本号是多少

* helloworld 控制文件

```
sunyongfeng@ubuntu:~/workshop/test$ cat helloworld_1.0-1/DEBIAN/control 
Package: helloworld
Version: 1.0-1
Section: base
Priority: optional
Architecture: amd64
Depends: mydep (>= 2.24)
Maintainer: Your Name <you@email.com>
Description: Hello World
 When you need some sunshine, just run this
  small program!
```

* mydep 控制文件

```
sunyongfeng@ubuntu:~/workshop/test$ cat mydep_2.24-1/DEBIAN/control 
Package: mydep
Version: 2.24-1
Section: base
Priority: optional
Architecture: amd64
Maintainer: Your Name <you@email.com>
Description: Hello World my dep
 When you need some sunshine my dep, just run this
  small program!
```

## 打包
通过命令 `dpkg-deb --build dir` 打包。

```
sunyongfeng@ubuntu:~/workshop/test$ dpkg-deb --build helloworld_1.0-1/
dpkg-deb: building package 'helloworld' in 'helloworld_1.0-1.deb'.
sunyongfeng@ubuntu:~/workshop/test$ dpkg-deb --build mydep_2.24-1/
dpkg-deb: building package 'mydep' in 'mydep_2.24-1.deb'
```

## 安装
### 依赖关系不成立，安装失败
如果先安装 helloworld，提示依赖的 mydep 未安装，失败。

```
sunyongfeng@ubuntu:~/workshop/test$ sudo dpkg -i helloworld_1.0-1.deb 
(Reading database ... 325288 files and directories currently installed.)
Preparing to unpack helloworld_1.0-1.deb ...
Unpacking helloworld (1.0-1) over (1.0-1) ...
dpkg: dependency problems prevent configuration of helloworld:
 helloworld depends on mydep (>= 2.24); however:
  Package mydep is not installed.

dpkg: error processing package helloworld (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 helloworld
```

### 依赖关系成立，安装成功

先安装 mydep。

```
sunyongfeng@ubuntu:~/workshop/test$ sudo dpkg -i mydep_2.24-1.deb 
Selecting previously unselected package mydep.
(Reading database ... 325288 files and directories currently installed.)
Preparing to unpack mydep_2.24-1.deb ...
Unpacking mydep (2.24-1) ...
Setting up mydep (2.24-1) ...
```

再安装 helloworld

```
sunyongfeng@ubuntu:~/workshop/test$ sudo dpkg -i helloworld_1.0-1.deb 
(Reading database ... 325289 files and directories currently installed.)
Preparing to unpack helloworld_1.0-1.deb ...
Unpacking helloworld (1.0-1) over (1.0-1) ...
Setting up helloworld (1.0-1) ...
```

## 卸载软件

如果先删除 mydep，会一般把依赖 mydep 的软件包都删除。

```
sunyongfeng@ubuntu:~/workshop/test$ sudo apt-get remove mydep
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be REMOVED:
  helloworld mydep
0 upgraded, 0 newly installed, 2 to remove and 278 not upgraded.
After this operation, 0 B of additional disk space will be used.
Do you want to continue? [Y/n] y
(Reading database ... 325288 files and directories currently installed.)
Removing helloworld (1.0-1) ...
Removing mydep (2.24-1) ...
dpkg: warning: while removing mydep, directory '/usr/local/bin' not empty so not removed
```

## 依赖关系数据库

deb 软件依赖关系数据库存放于 `/var/lib/dpkg/status`，纯文本文件。比如 mydep 和 helloworld 的信息如下。

```
sunyongfeng@ubuntu:~/workshop/test$ sudo vi /var/lib/dpkg/status

37582 Package: mydep                                                                                      
37583 Status: install ok installed                                                                        
37584 Priority: optional                                                                                  
37585 Section: base                                                                                       
37586 Maintainer: Your Name <you@email.com>                                                               
37587 Architecture: amd64                                                                                 
37588 Version: 2.24-1                                                                                     
37589 Description: Hello World my dep                                                                     
37590  When you need some sunshine my dep, just run this                                                  
37591   small program!  

42473 Package: helloworld                                                                                 
42474 Status: install ok installed                                                                        
42475 Priority: optional                                                                                  
42476 Section: base                                                                                       
42477 Maintainer: Your Name <you@email.com>                                                               
42478 Architecture: amd64                                                                                 
42479 Version: 1.0-1                                                                                      
42480 Depends: mydep (>= 2.24)                                                                            
42481 Description: Hello World                                                                            
42482  When you need some sunshine, just run this                                                         
42483   small program!   
```

### hack 依赖关系数据库
由于没有看源码确认，通过删除依赖关系数据库中 helloworld 依赖 mydep 的 Depends 行 `42480 Depends: mydep (>= 2.24) `。
通过删除 mydep 包，看 helloworld 会不会被连带删除确认依赖关系数据库是否生效。经验证，此时卸载 mydep 包，不会卸载 helloworld 包，与预期的相符。

```
sunyongfeng@ubuntu:~/workshop/test$ sudo vi /var/lib/dpkg/status

42462 Package: helloworld                                                                                 
42463 Status: install ok installed                                                                        
42464 Priority: optional                                                                                  
42465 Section: base                                                                                       
42466 Maintainer: Your Name <you@email.com>                                                               
42467 Architecture: amd64                                                                                 
42468 Version: 1.0-1                                                                                      
42469 Description: Hello World                                                                            
42470  When you need some sunshine, just run this                                                         
42471   small program! 

sunyongfeng@ubuntu:~/workshop/test$ sudo apt-get remove mydep
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be REMOVED:
  mydep
0 upgraded, 0 newly installed, 1 to remove and 278 not upgraded.
After this operation, 0 B of additional disk space will be used.
Do you want to continue? [Y/n] y
(Reading database ... 325288 files and directories currently installed.)
Removing mydep (2.24-1) ...
```

## 升级 mydep 版本，会不会提醒依赖它的 helloworld
答案是不会。

将 helloworld 的控制文件 Depends 行改为 `42480 Depends: mydep (= 2.24)`，升级 mydep 为版本 2.25-1，此时不会提醒 helloworld。但是重新安装 helloworld 会进一步提醒。

因此对被依赖的软件包，如果升级为不向前兼容的版本，只能以“死给你看”的形式“通知”依赖它的软件包。

```
sunyongfeng@ubuntu:~/workshop/test$ vi helloworld_1.0-1/DEBIAN/control
sunyongfeng@ubuntu:~/workshop/test$ rm helloworld_1.0-1.deb 
sunyongfeng@ubuntu:~/workshop/test$ dpkg-deb --build helloworld_1.0-1/
dpkg-deb: building package 'helloworld' in 'helloworld_1.0-1.deb'.
sunyongfeng@ubuntu:~/workshop/test$ sudo apt-get remove helloworld 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be REMOVED:
  helloworld
0 upgraded, 0 newly installed, 1 to remove and 278 not upgraded.
1 not fully installed or removed.
After this operation, 0 B of additional disk space will be used.
Do you want to continue? [Y/n] y
(Reading database ... 325288 files and directories currently installed.)
Removing helloworld (1.0-1) ...
sunyongfeng@ubuntu:~/workshop/test$ sudo dpkg -i helloworld_1.0-1.deb 
Selecting previously unselected package helloworld.
(Reading database ... 325288 files and directories currently installed.)
Preparing to unpack helloworld_1.0-1.deb ...
Unpacking helloworld (1.0-1) ...
Setting up helloworld (1.0-1) ...
sunyongfeng@ubuntu:~/workshop/test$ sudo dpkg -i mydep_2.24-1.deb 
dpkg: warning: downgrading mydep from 2.25-1 to 2.24-1
(Reading database ... 325288 files and directories currently installed.)
Preparing to unpack mydep_2.24-1.deb ...
Unpacking mydep (2.24-1) over (2.25-1) ...
Setting up mydep (2.24-1) ...

sunyongfeng@ubuntu:~/workshop/test$  sudo dpkg -i helloworld_1.0-1.deb 
(Reading database ... 325289 files and directories currently installed.)
Preparing to unpack helloworld_1.0-1.deb ...
Unpacking helloworld (1.0-1) over (1.0-1) ...
dpkg: dependency problems prevent configuration of helloworld:
 helloworld depends on mydep (= 2.24-1); however:
  Version of mydep on system is 2.25-1.

dpkg: error processing package helloworld (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 helloworld
```

## 参考资料
1. http://tldp.org/HOWTO/html_single/Debian-Binary-Package-Building-HOWTO/
2. https://www.debian.org/doc/manuals/debian-faq/ch-pkg_basics.en.html

> Written with [StackEdit](https://stackedit.io/).
