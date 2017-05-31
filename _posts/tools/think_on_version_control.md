title: 版本控制的思考
date: 2017-04-01 22:02:02
toc: true
tags: [版本控制, svn, git]
categories: tools
keywords: [svn, git, version control, 分支管理, branch management, master, develop, release, hotfix]
description: 关于版本控制分支管理的思考。
---

GIT 分支模型
--------------------

本文分支模型来自[《一个成功的 Git 分支模型》](http://blog.jobbole.com/81196/)。

文中采用 git 进行分支管理，亦可同时适用于 svn。

## 基础概念

![分支模型](http://ww3.sinaimg.cn/large/7cc829d3gw1en76ivwj9yj20vy16cdmb.jpg)

* master: 主分支，主要用来版本发布。
* develop：日常开发分支，该分支正常保存了开发的最新代码。
* feature：具体的功能开发分支，只与 develop 分支交互。
* release：release 分支可以认为是 master 分支的未测试版。比如说某一期的功能全部开发完成，那么就将 develop 分支合并到 release 分支，测试没有问题并且到了发布日期就合并到 master 分支，进行发布。
* hotfix：线上 bug 修复分支。

## 核心问题

* 项目太多，分支如何管理？
 + 产品组
 + 解决方案组
* 项目太赶，统一的分支应如何保证代码修订不会影响非本项目产品？
 + 听说另一个事业部的同学，改 3 行代码改出多个 bug。
 + 现实是，即使有统一的分支，为时效性，项目组只会测试本项目所涉及的产品，不会投入测试该统一分支的所有产品。
 
## 思考的一个分支管理模型

* 新项目分支；
 + 基于 develop 分支开 feature 分支，进行功能开发；
 + 功能开发结束后，如果功能没有被废弃，则将 feature 分支的修订同步回 develop 分支；
 + 从 develop 分支开出 release 分支，进行测试；
 + 如果测试没有问题，则合并到 master 分支，进行发布。
* 如果项目发布前，发现 bug，则基于 master 分支起 hot-fix 分支，hot-fix 分支解决完 bug 后，需要同步到 master 同时同步回 develop。
 + bug 定位由产品组/解决方案组定位；	
 + bug 解决由软件组解决；	
 + bug 同步由软件组同步。	
 + hot-fix 分支有期限，最多只维护半年。
* 如果是产品组/解决方案组的特殊功能需求，代码由产品组/解决方案组自行维护。

### 主分支

* master 分支
* develop 分支

约束：

* origin/master 这个分支上 HEAD 引用所指向的代码都是可发布的。
* origin/develop这个分支上HEAD引用所指向的代码总是反应了下一个版本所要交付特性的最新的代码变更。一些人管它叫“整合分支”。它也是自动构建系统执行构建命令的分支。

当 develop 分支上的代码达到一个稳定状态，并准备发布时，所有代码变更都应合并到 master 分支，并打上去发布版本号的 tag。

每次代码合并到master分支时，它就是一个人为定义的新的发布产品。理论上而言，在这我们应该非常严格，当master分支有新的提交时，我们应该使用Git的钩子脚本执行自动构建命令，然后将软件推送到生产环境的服务器中进行发布。

### 辅助性分支

紧邻 master 和 develop 分支，我们的开发模型采用了另外一种辅助性的分支，以帮助团队成员间的并行开发，特性的简单跟踪，产品的发布准备事宜，以及快速的解决线上问题。不同于主分支，这些辅助性分支往往只要有限的生命周期，因为他们最终会被删除。不同类型的分支包括：

* 特性分支，feature branches
* Release 分支
* Hotfix 分支

#### 特性分支

![feature branches](http://ww2.sinaimg.cn/mw690/7cc829d3gw1en76j016fwj207e0judge.jpg)

特性分支基于 develop 分支创建，最终合并入 develop 分支。

约束：

* 特性分支的命名，除了 master、develop、release-* 或 hotfix-* 以后，可以随便起名。
* 特性分支只存在开发者本地版本库，不在远程版本库。

特性分支(有时候也成主题分支)用于开发未来某个版本新的特性。当开始一个新特性的开发时，这个特性未来将发布于哪个目标版本，此刻我们是不得而知的。特性分支的本质特征就是只要特性还在开发，他就应该存在，但最终这些特性分支会被合并到develop分支(目的是在新版本中添加新的功能)或者被丢弃(它只是一个令人失望的试验)

#### 发布分支

发布分支基于 develop 分支创建，必须合并到 develop 分支和 master 分支。

约束：

* 命名为 release-*

Release分支用于支持一个新版本的发布。他们允许在最后时刻进行一些小修小改。甚至允许进行一些小bug的修改，为新版本的发布准要一些元数据(版本号，构建时间等)。通过在release分支完成这些工作，develop分支将会合并这些特性以备下一个大版本的发布。

从develop分支拉取新的release分支的时间点是当开发工作已经达到了新版本的期望值。至少在这个时间点，下一版本准备发布的所有目标特性必须已经合并到了develop分支。更远版本的目标特性不必合并会develop分支。这些特性必须等到个性分支创建后，才能合并回develop分支

在release分支创建好后，就会获取到一个分配好即将发布的版本号，不能更早，就在这个时间点。在此之前，develop分支代码反应出了下一版本的代码变更，但是到底下一版本是 0.3 还是 1.0，不是很明确，直到release分支被建立后一切都确定了。这些决定在release分支开始建立，项目版本号等项目规则出来后就会做出。

新的分支会存在一段时间，直到新版本最终发布。在这段时间里，bug的解决可以在这个分支进行(不要在develop分支进行)。此时是严禁添加新的大特性。这些修改必须合并回develop分支，之后就等待新版本的发布。

#### hotfix 分支

![hotfix分支](http://ww3.sinaimg.cn/mw690/7cc829d3gw1en76j1l9k0j20hk0no407.jpg)

Hotfix 分支基于 master 分支创建，必须合并回 develop 分支和 master 分支。

约束：

* 命名为 hotfix-*

Hotfix分支在某种程度上非常像release分支，他们都意味着为某个新版本发布做准备，并且都是预先不可知的。Hotfix分支是基于当前生产环境的产品的一个bug急需解决而必须创建的。当某个版本的产品有一个严重bug需要立即解决，Hotfix分支需要从master分支上该版本对应的tag上进行建立，因为这个tag标记了产品版本。

完成工作后，解决掉的bug代码需要合并回master分支，但同时也需要合并到develop分支，目的是保证在下一版中该bug已经被解决。这多么像release分支啊。

这里可能会有一些异常情况，**当一个release分支存在时，hotfix 分支需要合并到release 分支，而不是develop分支**。当release分支的使命完成后，合并回release分支的bugfix代码最终也会被合并到develop分支。(当develop分支急需解决这些bug，而等不到release分支的结束，你可以安全的将这些bugfix代码合并到develop分支，这样做也是可以的)。

