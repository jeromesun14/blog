title: BitBake 实用指南
date: 2016-10-21 01:27:20
toc: true
tags: [Linux, yocto]
categories: yocto
keywords: [Linux, yocto, bitbake, openswitch, openembedded, oe]
description: Bitbake 简明教程、实用指南。

[TOC]

0. 译者序
------------
本文译自 [A practical guide to BitBake](http://a4z.bitbucket.org/docs/BitBake/guide.html)。

如果你发现 bug、不清楚的章节、打印错误或其他建议，请邮件告知我，我的邮箱是 `sunnogo@gmail.com`。

> 注意：由于 task 和 recipe 是 BitBake 的基础概念。个人觉得翻译成任务和配方不免有误解之处，因此文中基本不对这两个词做翻译。类似的还有 configure。

1. 序言
---------

### 1.1 关于本教程
如果你阅读本教程，说明你已经知道 BitBake 是一种类似 make 的构建工具，主要用于 OpenEmbedded 和 Yocto 工程构建 Linux 发行版本。你可能也已经意识到 BitBake 的学习曲线有点陡，本文可让这个曲线变平缓一些。

本文不会告诉你 BitBake 的一切，但是会尝试解释使用 BitBake 时用到的一些基本功能。理解这些基础可帮助你开始写自己的 BitBake recipe。

### 1.2 本教程的目标
本教程展示如何创建一个最小工程，并一步步扩展，说明 BitBake 如何运作。

### 1.3 致谢
感谢 [Tritech](http://tritech.se/) 给我时间准备本文档。同时感谢大家在[问题跟踪站点](https://bitbucket.org/a4z/bitbakeguide/issues)报告的问题与打印错误。

### 1.4 反馈
如果你发现 bug、不清楚的章节、打印错误或其他建议， 请使用 issue tracker https://bitbucket.org/a4z/bitbakeguide/issues，不需要注册。
同时也可以使用本文底部的 Disqus 评论功能。

2. BitBake
-------------

### 2.1 什么是 BitBake
以下内容有助于理解 BitBake：
> 基本上，BitBake是一个Python程序，它由用户创建的配置驱动，可以为用户指定的目标执行用户创建的任务，即所谓的配方（recipes）。

#### 2.1.1 Config、tasks 与 recipes
通过一种 BitBake 领域特定语言写 Config、tasks 与 recipes，这种语言包含变量与可执行的 shell、python 代码。所以理论上，BitBake 可以执行代码，你也可以用 BitBake 做除构建软件之外的事情，但是并不推荐这么做。

BitBake 是一种构建软件的工具，因此有一些特殊的功能，比如可以定义依赖关系。BitBake 可以解决依赖关系，并将其任务以正确顺序运行。此外，构建软件包通常包含相同或相似的任务。比如常见的任务：下载源代码包，解压源代码，跑 configure，跑 make，或简单的输出 log。Bitbake 提供一种机制，可通过一种可配置的方式，抽象、封装和重用这个功能。

3. 配置 BitBake
---------------------
BitBake 可以从这里下载：https://github.com/openembedded/bitbake。选择一个版本的分支，并下载 zip。解压 zip 包，可找到一个 bitbake-$version 目录。

> 注意：本文使用的 Bitbake 版本是 bitbake-1.22，因此适合本教程的 bitbake 版本应该大于或等于1.22。
> 注意：译者使用 bitbake-1.27.0，因此文中样例为 1.27.0 版本 bitbake 样例。
> 提示：如果使用 Yocto，则不需要安装 BitBake，Yocto 源代码本身捆绑了 BitBake。Yocto 要求你 source 一个脚本，这个脚本和我们这里做的一样，安装 BitBake 到我们的环境中。

### 3.1 安装 BitBake
安装过程很简单：
* 添加 bitbake-$version/bin 目录到 PATH
* 添加 bitbake-$version/lib 目录到 PYTHONPATH

即执行：
```
export PATH=/path/to/bbtutor/bitbake/bin:$PATH
export PYTHONPATH=/path/to/bbtutor/bitbake/lib:$PYTHONPATH
```

这基本和 yocto init 脚本一致。yocto init 脚本同时也创建 build 目录，我们将在一会儿创建。

首先检测是不是一切正常、bitbake 是否安装成功。通过执行以下 bitbake 命令：
```
bitbake --version
```
运行结果应该类似：
```
BitBake Build Tool Core version 1.27.0
```

### 3.2 BitBake 文档
最实际的版本带有源代码。

在终端中，cd 到 bitbake-$version/doc 目录并执行以下命令，生成 doc/bitbake-user-manual/bitbake-user-manual.html。
```
make html DOC=bitbake-user-manual
```

这个文档可与本教程并行阅读，在读完本教程后也需要阅读该文档。

[yocto 工程文档](https://www.yoctoproject.org/documentation/current) 也有一个 bitbake 章节。

4. 创建工程
----------------
### 4.1 Bitbake 工程布局
通过 BitBake 工程通过 layers 目录与一个 build 目录组织，layer 目录包含配置文件和 meta data。

#### 4.1.1 Layer 目录
Layer 目录包含配置、任务和目标描述。常用 meta-'something' 命名 Layer 目录。

#### 4.1.2 Build 目录
Build 目录是 bitbake 命令被执行的地方。在这里，BitBake 期望能找到其初始配置文件，并将其生成的所有文件放在这个目录。

为了让 BitBake 运行时出现有任何错误，我们需要创建一个 build 目录和一个 layer 目录，并在此存放一些需要的配置文件。

### 4.2 最小工程
最小的配置看起来像这样：
```
bbTutorial/
├── build
│   ├── bitbake.lock
│   └── conf
│       └── bblayers.conf
└── meta-tutorial
    ├── classes
    │   └── base.bbclass
    └── conf
        ├── bitbake.conf
        └── layer.conf
```

需要创建这 4 个文件：
* bblayers.conf
* base.bbclass
* bitbake.conf
* layer.conf

#### 4.2.1 需要的配置文件
首先描述需要的文件，然后简要说明其内容。

`build/conf/bblayers.conf`，BitBake 在其工作目录（即 build 目录）期望找到的第一个文件。现在我们以以下内容创建一个 bblayers.conf：
```
BBPATH := "${TOPDIR}"
BBFILES ?= ""
BBLAYERS = "/path/to/meta-tutorial"
```

`meta-tutorial/conf/layer.conf`，每个 layer 需要一个 conf/layer.conf 文件。现在我们以以下内容创建它：
```
BBPATH .= ":${LAYERDIR}"
BBFILES += ""
```

`meta-tutorial/classes/base.bbclass`
`meta-tutorial/conf/bitbake.conf`
现在，这些文件可以从 BitBake 安装目录中获取。这些文件位于文件夹 bitbake-$version/conf 和 bitbake-$version/classes中。只需将它们复制到 tutorial 项目中。

#### 4.2.2 创建文件的一些注意事项
`build/conf/bblayers.conf`
* 添加当前目录到 BBPATH，TOPDIR 被 BitBake 设置为当前工作目录。
* 初始设置 BBFILES 变量为空，Recipes 在后面会添加。
* 添加我们 meta-tutorial 的路径到 BBLAYERS 变量。当 BitBake 开始执行时，它会搜索所有给定的 layer 目录，以便获得其他配置。

`meta-tutorial/conf/layer.conf`
* LAYERDIR 是 BitBake 传给其所加载 Layer 的变量。我们添加该路径到 BBPATH 变量。
* BBFILES 告诉 BitBake recipes 在哪，现在我们没有添加任何东西，但是一会儿我们会改变它。

> 注意事项。“.=” 和“+=” 以不添加空格、添加空格的方式，将追加值附给一个变量。

`conf/bitbake.conf`
conf/bibake.conf 包含 一系列我们讨论的变量。

`classes/base.bbclass`
一个 *.bbclass 文件包含共享功能。我们的 base.bbclass 包含一些我们一会儿使用的 log 函数，以及一个 buld 任务。
并不是很有用，但是 BitBake 有需求，因为如果没有任何具体业务时，BitBake 默认需求的。我们随后将改变此功能。

#### 4.2.3 BitBake 搜索路径
对于 BitBake 来讲，有许多 BBPATH 非法和文件路径。这说明如果我们告诉  BitBake 探索一些路径时，它会搜索 BBPATH。
我们添加  TOPDIR 和 LAYERDIR 到 BBPATH，放在 classes/base.bbclass 或 conf/bitbake.conf 中的任意一个。
当然，我们会添加 meta-tutorial 目录。
编译目录不应含有通用文件。只有像 local.conf 对实际编译是有效的，后面我们会用到 local.conf。

### 第一次运行
创建上述四个配置文件后，在终端 cd 到 build 目录，这是我们的工作目录。我们一直在 build 目录运行 bitbake 命令，以便 bitbake 可以找到相应的 conf/bblayers.conf 文件。

现在，**在 build 目录**，不带任何参数运行 bitbake 命令：
```
bitbake
```

如果先前的步骤正确，则控制台会输出：
```
Nothing to do.  Use 'bitbake world' to build everything, or run 'bitbake --help' for usage information.
```
这没什么用，但是一个好的开始。

这里介绍一个很有用的命令标志：输出一些 debug 信息。
执行 `bitbake -vDD`，然后查看其输出，它告诉我们大量关于 BitBake 如何动作的信息。

```
DEBUG: Found bblayers.conf (~/bbTutorial/build/conf/bblayers.conf)
DEBUG: LOAD ~/bbTutorial/build/conf/bblayers.conf
DEBUG: Adding layer ~/bbTutorial/meta-tutorial
DEBUG: LOAD ~/bbTutorial/meta-tutorial/conf/layer.conf
DEBUG: LOAD ~/bbTutorial/meta-tutorial/conf/bitbake.conf
DEBUG: BB configuration INHERITs:0: inheriting ~/bbTutorial/meta-tutorial/classes/base.bbclass
DEBUG: BB ~/bbTutorial/meta-tutorial/classes/base.bbclass: handle(data, include)
DEBUG: LOAD ~/bbTutorial/meta-tutorial/classes/base.bbclass
DEBUG: Clearing SRCREV cache due to cache policy of: clear
DEBUG: Using cache in '~/bbTutorial/build/tmp/cache/local_file_checksum_cache.dat'
DEBUG: Using cache in '~/bbTutorial/build/tmp/cache/bb_codeparser.dat'
```

你在注意到 BitBake 创建了一个 bitbake.log 文件和一个 tmp 目录？
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ ls
bitbake.lock  conf  tmp
```

> 提示，所有的样例代码都可从 https://bitbucket.org/a4z/bitbakeguide 获取。本样例在 ch04。

5. 第一个 recipe
------------------
BitBake 需要 recipes 定义要做些什么，现在这里什么都没有。
我们可以通过 `bitbake -s` 确认运行时什么也没做：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake -s
ERROR: no recipe files to build, check your BBPATH and BBFILES?

Summary: There was 1 ERROR message shown, returning a non-zero exit code.
NOTE: Not using a cache. Set CACHE = <directory> to enable.
Recipe Name                                    Latest Version         Preferred Version
===========                                    ==============         =================

sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ 
```

这告诉我们两个信息：
1. 没有定义任何 cache；
2. BitBake 真的没事可做，只显示了一个空的 recipe 列表

### 5.1 cache 位置
BitBake 缓存 meta data 在一个目录，即 cache 目录。这会帮助加速后面执行的命令。

我们可通过简单添加一个变量到 bitbake.conf 文件，解决 cache 找不到的问题。因此，我们编辑 meta-tutorial/conf/bitbake.conf 文件，并在底部添加：

```
CACHE = "${TMPDIR}/cache/default"
```

添加后运行 `bitbake -s` 的结果：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake -s
ERROR: no recipe files to build, check your BBPATH and BBFILES?

Summary: There was 1 ERROR message shown, returning a non-zero exit code.
```

> 注意：在实现项目中，比如 Yocto，这些变量已经设置好，我们不用关心。通常 cache 路径由不同的变量组成，在名称中包含实际的构建配置，如 debug 或 release。

下一步是添加一个 recipe，需要两个步骤：
1. 使 bitbake 可以找到 recipes
2. 写第一个 recipe

### 5.2 添加一个 recipe 到 tutorial layer
BitBake 需要知道一个 layer 提供哪些 recipes，可通过编辑 meta-tutorial/conf/layer.conf 文件，使用通配符告诉 BitBake 加载所有的 recipe：
```
BBPATH .= ":${LAYERDIR}"
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb"
```
现在可以使用先前在 build/conf/bblayers.conf 定义的变量。recipe 文件的扩展名是 .bb，如果我们通过通配符的方式，只用一行就可以告诉 BitBake 加载所有 recipes。

通常 recipes 有自己的目录，并以 groups 的形式收集在一起，也就是说把有关联的 recipes 放在同一个目录。

> 注意：通常使用 recipes-'group' 命令这些目录，这里 group 名表示一个 category 或一些程序。

现在 BitBake 已经知道从哪找 recipe，我们可以开始添加第一个 recipe 了。

按通常的做法，我们创建目录 meta-tutorial/recipes-tutorial/first，并在此创建第一个 recipe。 Recipe 文件也有通用的命名方法：{recipe}_{version}.bb

### 5.3 创建第一个 recipe 和 task
我们的第一个 recipe 只打印一些 log 信息。将它放在 tutorial group，版本为 0.1。所以我们的第一个 recipe 是：
`meta-tutorial/recipes-tutorial/first/first_0.1.bb`
```
DESCRIPTION = "I am the first recipe"
PR = "r1"
do_build () {
  echo "first: some shell script running as build"
}
```

* task do_build 覆盖 base.bbclass 中的全局 build task。
* PR 是内部修订数据，在每次修订后应被更新。
* 设置 description 可解释该 recipe 的用途。

如果上面都做对了，可以通过 `bitbake -s` 列出可用的 recipes。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake -s
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 1 .bb files complete (0 cached, 1 parsed). 1 targets, 0 skipped, 0 masked, 0 errors.
Recipe Name                                    Latest Version         Preferred Version
===========                                    ==============         =================

first                                                 :0.1-r1                          
```

然后就可以执行 `bitbake first` 编译 first 组件。
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake first
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 1 .bb files complete (0 cached, 1 parsed). 1 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 1 tasks of which 0 didn't need to be rerun and all succeeded.
```

现在检查  tmp/work/first-0.1-r1/temp 目录，里面有一些有趣的文件：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ ls -al tmp/work/first-0.1-r1/temp/
total 20
drwxrwxr-x 2 sunyongfeng sunyongfeng 4096 10月 20 11:19 .
drwxrwxr-x 3 sunyongfeng sunyongfeng 4096 10月 20 11:19 ..
lrwxrwxrwx 1 sunyongfeng sunyongfeng   18 10月 20 11:19 log.do_build -> log.do_build.17314
-rw-rw-r-- 1 sunyongfeng sunyongfeng  123 10月 20 11:19 log.do_build.17314
-rw-rw-r-- 1 sunyongfeng sunyongfeng   37 10月 20 11:19 log.task_order
lrwxrwxrwx 1 sunyongfeng sunyongfeng   18 10月 20 11:19 run.do_build -> run.do_build.17314
-rwxrwxr-x 1 sunyongfeng sunyongfeng  909 10月 20 11:19 run.do_build.17314

sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/first-0.1-r1/temp/log.do_build.17314 
DEBUG: Executing shell function do_build
first: some shell script running as build
DEBUG: Shell function do_build finished
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/first-0.1-r1/temp/log.task_order 
do_build (17314): log.do_build.17314
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/first-0.1-r1/temp/run.do_build
#!/bin/sh

# Emit a useful diagnostic if something fails:
bb_exit_handler() {
    ret=$?
    case $ret in
    0)  ;;
    *)  case $BASH_VERSION in
        "")   echo "WARNING: exit code $ret from a shell command.";;
        *)    echo "WARNING: ${BASH_SOURCE[0]}:${BASH_LINENO[0]} exit $ret from
  "$BASH_COMMAND"";;
        esac
        exit $ret
    esac
}
trap 'bb_exit_handler' 0
set -e
export HOME="/home/sunyongfeng"
export SHELL="/bin/bash"
export LOGNAME="sunyongfeng"
export USER="sunyongfeng"
export PATH="/home/sunyongfeng/ops-build.test/yocto/poky/scripts:/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
export TERM="linux"
do_build() {
  echo "first: some shell script running as build"

}

cd '/home/sunyongfeng/workshop/test/tutorial/build'
do_build

# cleanup
ret=$?
trap '' 0
exit $?
```

6. Classes 和 functions
-------------------------------
下一步将：
* 添加一个 class
* 添加一个使用 class 的 recipe
* 探索 functions

### 6.1 创建 mybuild class
创建一个不同的 build 函数，并共享。先在 tutorial layer 创建 class，如 `meta-tutorial/classes/mybuild.bbclass`：
```
addtask build
mybuild_do_build () {
 
  echo "running mybuild_do_build."
 
}
 
EXPORT_FUNCTIONS do_build
```
在 base.class 中，我们添加了一个 build task，它也是一个简单的 shell 函数。mybuild_do 前缀的依据是 class 中 task 定义的规范 classname_do_functionname。

EXPORT_FUNCTIONS 使该 build 函数可被这个 class 的使用者使用，如果不添加这行，则它不会覆盖 base class 中的 build 函数。

现在，已可在第二个 recipe 中使用这个 class。

### 6.2 在第二个 recipe 中使用 myclass
这里添加一个小目标，在 build 任务前先运行一个 patch 函数，这里需要一些 python 的用法。

依据 bitbake 的命名规范，我们添加一个新的 recipe 目录，并在该目录内添加一个 recipe 文件 `meta-tutorial/recipes-tutorial/second/second_1.0.bb`：

```
DESCRIPTION = "I am he second recipe"
PR = "r1"                       (1)
inherit mybuild                 (2)
 
def pyfunc(o):                  (3)
    print dir (o)
 
python do_mypatch () {          (4)
  bb.note ("runnin mypatch")
  pyfunc(d)                     (5)
}
 
addtask mypatch before do_build (6)
```

1. 像 first recipe 那样定义 DESCRIPTION 和 PR；
2. 继承 mybuild class，让 myclass_do_build 成为默认 build task；
3. 纯 python 函数  pyfunc 获取一些参数，并根据该入参运行 python dir 函数；
4. bitbake python 函数  my_patch 添加并注册成一个 task，该 task 要在 build 函数前执行。
5. mypatch 函数调用 pyfunc 函数，并传入全局 bitbake 变量 d。d (datastore) 由 bitbake 定义，并一直可用。
6. mypatch 函数被注册成一个 task，并要求在 build 函数前执行。

这就是一个使用 python 函数的样例。

> 注意：函数部分的内容在 bitbake 手册 3.4 节。

### 6.3 探索 recipes 和 tasks
现在我们有两个 recipes 可用，可探索一些新的 bitbake 命令选项。我们可以获取BitBake 运行时 recipes 及其 tasks、控制过程的信息。

#### 6.3.1 显示 recipes 和 tasks 列表

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake -s
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 2 .bb files complete (0 cached, 2 parsed). 2 targets, 0 skipped, 0 masked, 0 errors.
Recipe Name                                    Latest Version         Preferred Version
===========                                    ==============         =================

first                                                 :0.1-r1                          
second                                                :1.0-r1 
```
如果想看某个 recipe 提供哪些 tasks，可以通过  `bitbake -c listtasks recipe_name` 查看：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake -c listtasks second
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 2 .bb files complete (0 cached, 2 parsed). 2 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
do_showdata
do_build
do_mypatch
do_listtasks
NOTE: Tasks Summary: Attempted 1 tasks of which 0 didn't need to be rerun and all succeeded.
```
### 6.4 执行 tasks 或完整构建
有些选项可在 recipes 执行 builds 或特定任务时使用。
* 构建一个 recipe。使用 `bitbade recipe-name` 执行该 recipe 的所有 tasks。
* 执行一个 task。使用 `bitbake -c your-task recipe-name` 只运行 recipe 中的某个 task。
* 构建所有 recipe。使用 `bitbake world` 运行所有 recipes 的所有  tasks。

可以玩玩这些命令，看会出现什么。

#### 6.4.1 确认构建过程中的 log
Bitbake 创建一个 tmp/work 目录存放所有的 log 文件。这些 log 文件包含一些有趣的信息，值得一学。第一次执行完  bitbake world ，其输出为：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake world
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 2 .bb files complete (0 cached, 2 parsed). 2 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 3 tasks of which 0 didn't need to be rerun and all succeeded.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ tree tmp/work/           
tmp/work/
├── first-0.1-r1
│   └── temp
│       ├── log.do_build -> log.do_build.17657
│       ├── log.do_build.17657
│       ├── log.task_order
│       ├── run.do_build -> run.do_build.17657
│       └── run.do_build.17657
└── second-1.0-r1
    ├── second-1.0
    └── temp
        ├── log.do_build -> log.do_build.17659
        ├── log.do_build.17659
        ├── log.do_mypatch -> log.do_mypatch.17656
        ├── log.do_mypatch.17656
        ├── log.task_order
        ├── run.do_build -> run.do_build.17659
        ├── run.do_build.17659
        ├── run.do_mypatch -> run.do_mypatch.17656
        └── run.do_mypatch.17656
```
这些 log 文件包含很多有用的信息，比如 BitBake 如何运行，执行 tasks 输出了什么。

7. BitBake layers
----------------------
典型 BitBake 工程包含多个 layer。通常 layer 包含一个特定主题，比如基础系统、图像系统等。一些工程可以包括不止一个构建目标，每个目标由不同的 layers 组成。例如，构建一个带 GUI 组件或不带 GUI 组件的 Linux 发行版本。

Layers 可以被使用、扩展、配置，也可能部分覆盖已有的 layers。这很重要，因为它允许根据实际要求重用或自定义。

多个 layers 共同动作是通用的例子，因此我们会添加一个额外的层次到工程。

### 7.1 添加一个 layer
通过以下步骤添加一个新的 layer：
1. 创建一个新的 layer 目录
2. 创建 layer 配置
3. 告诉 BitBake 有新的 layer
4. 添加 recipes 到 layer

#### 7.1.1 添加新的 layer 目录
创建一个新的目录叫 meta-two：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial$ ls
build  meta-tutorial  meta-two
```

#### 7.1.2 配置新 layer
添加 meta-two/conf/layer.conf 文件，该文件和 tutorial layer 的一样：
```
BBPATH .= ":${LAYERDIR}"
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb"
```

#### 7.1.3 告诉 BitBake 有新的 layer
编辑 build/conf/bblayers.conf，扩展 BBLAYERS 变量：
```
BBLAYERS = " \
  ${TOPDIR}/../meta-tutorial \
  ${TOPDIR}/../meta-two \
"
```

### bitbake-layer 命令
通过 `bitbake-layer` 命令检查新 layer 配置。
首先使用 show-layers 选项，显示该工程的 layers、layers 路径和优先级。这里优先级都是 0，后面会尝试改一下。
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake-layers show-layers
layer                 path                                      priority
==========================================================================
meta-tutorial         /home/sunyongfeng/workshop/test/tutorial/build/../meta-tutorial  0
meta-two              /home/sunyongfeng/workshop/test/tutorial/build/../meta-two  0
```

bitbake-layers 命令还有其他有用的选项，可通过 -h 选项显示。
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake-layers -h
usage: bitbake-layers [-h] [-d] [-q] <subcommand> ...

BitBake layers utility

optional arguments:
  -h, --help            show this help message and exit
  -d, --debug           Enable debug output
  -q, --quiet           Print only errors

subcommands:
  <subcommand>
    show-layers         show current configured layers
    add-layer           Add a layer to bblayers.conf
    remove-layer        Remove a layer from bblayers.conf
    show-overlayed      list overlayed recipes (where the same recipe exists
                        in another layer)
    show-recipes        list available recipes, showing the layer they are
                        provided by
    show-appends        list bbappend files and recipe files they apply to
    flatten             flatten layer configuration into a separate output
                        directory.
    show-cross-depends  Show dependencies between recipes that cross layer
                        boundaries.
    layerindex-fetch    Fetches a layer from a layer index along with its
                        dependent layers, and adds them to conf/bblayers.conf.
    layerindex-show-depends
                        Find layer dependencies from layer index.

Use bitbake-layers <subcommand> --help to get help on a specific command
```

### 7.3 扩展 layer 配置
在 layer 的 layer.conf 文件中，定义优化级和其他配置值。为配置 layer 的优先级，需要添加新的定义到已有的 layer.conf。以 meta-tutorial/conf/layer.conf 开始，添加：
```                                                                                     
# append layer name to list of configured layers                                                       
BBFILE_COLLECTIONS += "tutorial"                                                                       
# and use name as suffix for other properties                                                          
BBFILE_PATTERN_tutorial = "^${LAYERDIR}/"                                                              
BBFILE_PRIORITY_tutorial = "5" 
```
使用的变量在 BitBake 使用手册有很好的说明，这里不重复。

模式应是清楚的，这里定义 layer 名，并使用这个名字做为其他变量的后缀。这种在 BitBake 变量名中使用用户定义的域后缀机制，在 BitBake 的很多地方可以看到。

同样的，修改 meta-two/conf/layer.conf：
```
# append layer name to list of configured layers                                                       
BBFILE_COLLECTIONS += "tutorial"                                                                       
# and use name as suffix for other properties                                                          
BBFILE_PATTERN_tutorial = "^${LAYERDIR}/"                                                              
BBFILE_PRIORITY_tutorial = "5" 
```

如果此时运行 bitbake-layers show-layers，结果是：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake-layers show-layers     
layer                 path                                      priority
==========================================================================
meta-tutorial         /home/sunyongfeng/workshop/test/tutorial/build/../meta-tutorial  5
meta-two              /home/sunyongfeng/workshop/test/tutorial/build/../meta-two  5
```

8. 共享和重用配置
-------------------------
截止目前，我们使用 classes 和 config 文件封装配置和 tasks。但是还有更多的方法重用和扩展 tasks 和配置：
* class 继承
* bbappend 文件
* include 文件

为说明如何使用这些方法，我们将添加 class 到 layer-two，新的 class 将介绍一个 configure-build 链并使用 class 继承重用现存的 mybuild class。然后在新的 recipe 中使用这个新 class，最后通过 append 方法扩展现有的 recipe。

### 8.1 class 继承
为实现 configure-build 链，这里创建一个 class，该 class 继承 mybuild，并简单添加一个 configure task，让 build task 依赖 configure task。

`meta-two/classes/confbuild.bbclass`：
```
inherit mybuild                            (1)
 
confbuild_do_configure () {                (2)
 
  echo "running configbuild_do_configure."
 
}
 
addtask do_configure before do_build       (3)
 
EXPORT_FUNCTIONS do_configure              (4)
```

1. 以 mybuild class 为基础；
2. 创建新的函数；
3. 定义函数的顺序，configre 在 build 之前；
4. export 刚创建的函数使之可用。

然后创建 third recipe 使用 confbuild class。
`meta-two/recipes-base/third_01.bb`：
```
DESCRIPTION = "I am the third recipe"
PR = "r1"
inherit confbuild
```
这时运行 `bitabke third` 会执行 configure 和 build task。
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake third
Parsing recipes: 100% |######################################################################################################################################| Time: 00:00:00
Parsing of 3 .bb files complete (0 cached, 3 parsed). 3 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 2 tasks of which 0 didn't need to be rerun and all succeeded.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/third-01-r1/temp/
log.do_build            log.do_configure        log.task_order          run.do_build.19728      run.do_configure.19726  
log.do_build.19728      log.do_configure.19726  run.do_build            run.do_configure        
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/third-01-r1/temp/log.task_order 
do_configure (19726): log.do_configure.19726
do_build (19728): log.do_build.19728
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/third-01-r1/temp/log.do_configure
DEBUG: Executing shell function do_configure
running configbuild_do_configure.
DEBUG: Shell function do_configure finished
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/third-01-r1/temp/log.do_build
DEBUG: Executing shell function do_build
running mybuild_do_build.
DEBUG: Shell function do_build finished
```

### 8.2 bbappend 文件
append 文件可以添加函数到已有 class 中，而不需要创建一个新 class。它向同名 class 添加 append 文件的文本。需要设置 layer 配置，才能加载到对应的 append 文件。因此需要改变 layer 的配置，添加加载 *.bbappend 文件的配置到  BBFILES 变量。例如：
`meta-two/conf/layer.conf`：
```
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
${LAYERDIR}/recipes-*/*/*.bbappend"
```

现在扩展已有的 first recipe，让它在 build task 前先运行一个 patch 函数。为做对比，将对应的 recipe 和 append 文件放到 meta-two/recipes-base/first 目录。
`meta-two/recipes-base/first/first_0.1.bbappend`：
```
python do_patch () {
  bb.note ("first:do_patch")
}
 
addtask patch before do_build
```

此时若列出 first recipe 的 task 列表，可以看到 patch task。运行 bitbake first 可看到运行了 patch 和 build。

添加前：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake -c listtasks first
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 3 .bb files complete (0 cached, 3 parsed). 3 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
do_showdata
do_build
do_listtasks
NOTE: Tasks Summary: Attempted 1 tasks of which 0 didn't need to be rerun and all succeeded.
```

添加 append 文件后：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake -c listtasks first
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 3 .bb files complete (0 cached, 3 parsed). 3 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
do_showdata
do_build
do_listtasks
do_patch
NOTE: Tasks Summary: Attempted 1 tasks of which 0 didn't need to be rerun and all succeeded.
```

运行 bitbake first 结果：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake first
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 3 .bb files complete (0 cached, 3 parsed). 3 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 2 tasks of which 0 didn't need to be rerun and all succeeded.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/first-0.1-r1/temp/
log.do_build            log.do_listtasks.20111  run.do_build            run.do_listtasks.20111
log.do_build.20152      log.do_patch            run.do_build.20152      run.do_patch
log.do_listtasks        log.do_patch.20151      run.do_listtasks        run.do_patch.20151
log.do_listtasks.20001  log.task_order          run.do_listtasks.20001  
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/first-0.1-r1/temp/log.task_order 
do_listtasks (20001): log.do_listtasks.20001
do_listtasks (20111): log.do_listtasks.20111
do_patch (20151): log.do_patch.20151
do_build (20152): log.do_build.20152
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ 
```

> 提示：如果你愿意，现在就可以构建一个 recipe，使用 confbuild class 和一个 append 文件，运行 patch、configure 和 build 任务。

### 8.3 include 文件
BitBake 有两种指令引用文件：
* `include filename`，这是一种可选引用，如果 filename 找不到，不会有 error 产生；
* `require filename`，如果 filename 没找到，会产生 error。

值得一提的是，include 和 require 都是在 BBPATH 中指定的目录查找 filename。

#### 8.3.1 添加 local.conf 用于引用文件
BitBake 工程通常使用 bitbake.conf 引用一个位于 build 目录内的 local.conf 文件。local.conf 文件可能包含一些当前构建目标相关的特殊设置。典型的样例是 Yocto 的设置。

这里模仿 local.conf 的典型应用，让 bitbake.conf require 引用 local.conf，添加以下内容到 meta-tutorial/conf/bitbake.conf：
```
require local.conf
include conf/might_exist.conf
```
如果此时执行构建命令，BitBake 会产生类似以下的错误信息：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake world
ERROR: Traceback (most recent call last):
  File "/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/lib/bb/cookerdata.py", line 175, in wrapped
    return func(fn, *args)
  File "/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/lib/bb/cookerdata.py", line 185, in parse_config_file
    return bb.parse.handle(fn, data, include)
  File "/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/lib/bb/parse/__init__.py", line 107, in handle
    return h['handle'](fn, data, include)
  File "/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/lib/bb/parse/parse_py/ConfHandler.py", line 148, in handle
    statements.eval(data)
  File "/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/lib/bb/parse/ast.py", line 39, in eval
    statement.eval(data)
  File "/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/lib/bb/parse/ast.py", line 61, in eval
    bb.parse.ConfHandler.include(self.filename, s, self.lineno, data, "include required")
  File "/home/sunyongfeng/ops-build.test/yocto/poky/bitbake/lib/bb/parse/parse_py/ConfHandler.py", line 98, in include
    raise ParseError("Could not %(error_out)s file %(fn)s" % vars(), parentfn, lineno)
ParseError: ParseError at /home/sunyongfeng/workshop/test/tutorial/build/../meta-tutorial/conf/bitbake.conf:53: Could not include required file local.conf

ERROR: Unable to parse conf/bitbake.conf: ParseError at /home/sunyongfeng/workshop/test/tutorial/build/../meta-tutorial/conf/bitbake.conf:53: Could not include required file local.conf
```
添加一个 local.conf 文件到 build 目录可解决此问题。注意 include 语句包含的文件可有可无。

9. 使用变量
----------------
可定义变量并在 recipes 中使用，让 BitBake 具有很强的灵活性。可将可配置部分使用变量的方式编写 recipe，这种 recipe 的用户可以给出那些将由 recipe 使用的变量值。一个典型的例子是给 recipe 传递额外的配置或标志。通过正确使用变量，不需要编辑和更改 recipe，因为某些函数只需要一些特殊的参数。

### 9.1 全局变量
全局变量可以通过使用者设置，recipe 可以使用。

#### 9.1.1 定义全局变量
刚才已经创建一个空的 local.conf，现在在这个文件加一些变量。比如添加一行：
```
MYVAR="hello from MYVAR"
```

#### 9.1.2 访问全局变量
可以在 recipes 或 classes 中访问 MYVAR 变量。这里创建一个新的 recipes 组 recipes-vars，及一个 recipe myvar。
`meta-two/recipes-vars/myvar/myvar_0.1.bb`：
```
DESCRIPTION = "Show access to global MYVAR"
PR = "r1"
 
do_build(){
  echo "myvar_sh: ${MYVAR}"                        (1)
}
 
python do_myvar_py () {
  print "myvar_py:" + d.getVar('MYVAR', True)      (2)
}
 
addtask myvar_py before do_build
```
1. 在类 bash 语法中访问变量；
2. 通过全局数据存储访问变量。

现在运行 `bitbake myvar`，检查 tmp 目录的输出，则可以看到我们确实访问了全局 MYVAR 变量。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake myvar
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 4 .bb files complete (0 cached, 4 parsed). 4 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 2 tasks of which 0 didn't need to be rerun and all succeeded.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/myvar-0.1-r1/temp/log.task_order 
do_myvar_py (4595): log.do_myvar_py.4595
do_build (4596): log.do_build.4596
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/myvar-0.1-r1/temp/log.do_myvar_py
DEBUG: Executing python function do_myvar_py
myvar_py:hello from MYVAR
DEBUG: Python function do_myvar_py finished
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/myvar-0.1-r1/temp/log.do_build
DEBUG: Executing shell function do_build
myvar_sh: hello from MYVAR
DEBUG: Shell function do_build finished
```
### 9.2 本地变量
典型的 recipe 只包含一些本地变量，这些变量用于其继承的 classes 中的函数设置。

先创建 `meta-two/classes/varbuild.bbclass`：
```
varbuild_do_build () {
  echo "build with args: ${BUILDARGS}"
}
 
addtask build
 
EXPORT_FUNCTIONS do_build
```

然后在 `meta-two/recipes-vars/varbuld/varbuild_0.1.bb` 中使用：
```
DESCRIPTION = "Demonstrate variable usage \
  for setting up a class task"
PR = "r1"
 
BUILDARGS = "my build arguments"
 
inherit varbuild
```

运行 `bitbake varbuild`，输出的 log 显示 build 任务使用了 recipe 设置的变量值。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ bitbake varbuild
Parsing recipes: 100% |################################################################################| Time: 00:00:00
Parsing of 5 .bb files complete (0 cached, 5 parsed). 5 targets, 0 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
NOTE: Preparing RunQueue
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 1 tasks of which 0 didn't need to be rerun and all succeeded.
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/varbuild-0.1-r1/temp/log.task_order 
do_build (4760): log.do_build.4760
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ cat tmp/work/varbuild-0.1-r1/temp/log.do_build
DEBUG: Executing shell function do_build
build with args: my build arguments
DEBUG: Shell function do_build finished
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial/build$ 
```

这是使用 BitBake的典型方法。通用 task 由 class 定义，比如下载源代码、configure、make 和其他操作，recipe 设置这些 task 所需要的变量。

10. 附目录树
-----------------
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/test/tutorial$ tree ./*
./build
├── bitbake.lock
├── conf
│   └── bblayers.conf
└── local.conf
./meta-tutorial
├── classes
│   ├── base.bbclass
│   └── mybuild.bbclass
├── conf
│   ├── bitbake.conf
│   └── layer.conf
└── recipes-tutorial
    ├── first
    │   └── first_0.1.bb
    └── second
        └── second_1.0.bb
./meta-two
├── classes
│   ├── confbuild.bbclass
│   └── varbuild.bbclass
├── conf
│   ├── bitbake.conf
│   └── layer.conf
├── recipes-base
│   ├── first
│   │   └── first_0.1.bbappend
│   └── third
│       └── third_01.bb
└── recipes-vars
    ├── myvar
    │   └── myvar_0.1.bb
    └── varbuild
        └── varbuild_0.1.bb

14 directories, 17 files
```

11. 总结
-----------
以上是本教程的所有内容，感谢你一直看到这里，希望你喜欢。
学习完教程，你应该对 BitBake 的基本概念有基本理解。本教程涉及的内容有：
* BitBake 是一个执行 python 和 shell 脚本的引擎；
* 常见的 BitBake 工程设计与一些默认文件的位置；
* BitBake 使用的 5 种文件类型（.bb，.bbclass，.bbappend，.conf 和 include 文件）；
* BitBake 函数和 task，说明如何组织、分组和调用它们；
* BitBake 变量及其基本使用方法 。

熟悉这些内容后，希望你可以开始使用类似 Yocto 的工程，并继续深入理解。
