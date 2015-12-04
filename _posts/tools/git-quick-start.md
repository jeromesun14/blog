title: Git quick start
date: 2015-04-27 22:48:18
toc: true
tags: [版本控制, git]
categories: tools
keywords: [git, command, version control]
description: 本文描述git的基础使用方法、基本概念。
---
git - the stupid content tracker

这里讲GIT命令行客户端工具如何使用，不涉及GIT服务器如何部署。

# 入门操作 #
本节内容完全引用自[《git - 简易指南》][1]，作者：罗杰·杜德勒。如该文一开始所述：“助你开始使用 git 的简易指南，木有高深内容，;)”。

这个网站很酷炫，建议直接点进去看。
## 下载代码（检出仓库） ##
执行如下命令以创建一个本地仓库的克隆版本：
```
git clone /path/to/repository
```
如果是远端服务器上的仓库，你的命令会是这个样子：
```
git clone username@host:/path/to/repository
```

<!--more-->

## 工作流 ##
你的本地仓库由 git 维护的三棵“树”组成。第一个是你的 工作目录，它持有实际文件；第二个是 缓存区（Index），它像个缓存区域，临时保存你的改动；最后是 HEAD，指向你最近一次提交后的结果。
![trees][2]

## 添加与提交 ##
你可以计划改动（把它们添加到缓存区），使用如下命令：
```
git add <filename>
git add *
```
这是 git 基本工作流程的第一步；使用如下命令以实际提交改动：
```
git commit -m "代码提交信息"
```
现在，你的改动已经提交到了 HEAD，但是还没到你的远端仓库。

## 推送改动 ##
推送改动
你的改动现在已经在本地仓库的 HEAD 中了。执行如下命令以将这些改动提交到远端仓库：
```
git push origin master
```
可以把 master 换成你想要推送的任何分支。 

如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：
```
git remote add origin <server>
```
如此你就能够将你的改动推送到所添加的服务器上去了。

## 分支 ##
分支是用来将特性开发绝缘开来的。在你创建仓库的时候，master 是“默认的”。在其他分支上进行开发，完成后再将它们合并到主分支上。

![branches][3]

创建一个叫做“feature_x”的分支，并切换过去：
```
git checkout -b feature_x
```
切换回主分支：
```
git checkout master
```
再把新建的分支删掉：
```
git branch -d feature_x
```
除非你将分支推送到远端仓库，不然该分支就是 不为他人所见的：
```
git push origin <branch>
```

## 更新与合并 ##
要更新你的本地仓库至最新改动，执行：
```
git pull
```
以在你的工作目录中 获取（fetch） 并 合并（merge） 远端的改动。
要合并其他分支到你的当前分支（例如 master），执行：
```
git merge <branch>
```
两种情况下，git 都会尝试去自动合并改动。不幸的是，自动合并并非次次都能成功，并可能导致 冲突（conflicts）。 这时候就需要你修改这些文件来人肉合并这些 冲突（conflicts） 了。改完之后，你需要执行如下命令以将它们标记为合并成功：
```
git add <filename>
```
在合并改动之前，也可以使用如下命令查看：
```
git diff <source_branch> <target_branch>
```

## 标签 ##
在软件发布时创建标签，是被推荐的。这是个旧有概念，在 SVN 中也有。可以执行如下命令以创建一个叫做 1.0.0 的标签：
```
git tag 1.0.0 1b2e1d63ff
```
1b2e1d63ff 是你想要标记的提交 ID 的前 10 位字符。使用如下命令获取提交 ID：
```
git log
```
你也可以用该提交 ID 的少一些的前几位，只要它是唯一的。

## 替换本地改动 ##
替换本地改动
假如你做错事（自然，这是不可能的），你可以使用如下命令替换掉本地改动：
```
git checkout -- <filename>
```
此命令会使用 HEAD 中的最新内容替换掉你的工作目录中的文件。已添加到缓存区的改动，以及新文件，都不受影响。

假如你想要丢弃你所有的本地改动与提交，可以到服务器上获取最新的版本并将你本地主分支指向到它：
```
git fetch origin
git reset --hard origin/master
```

# GIT与SVN的差异 #
本节大部分内容引用自[《GIT和SVN之间的五个基本区别》][4]：

## 仓库管理方式 ##
GIT是分布式的，SVN是集中式的。GIT代码clone下来之后，就是一个完整的仓库，本地不管在线或离线，都可以提交修订（到本地仓库）、查看历史版本、创建分支等。而SVN是集中式管理，如果离线，则几乎无法进行任何工作，不能查看历史版本、不能查看log、不能提交修订...

现在还不理解这段话
>同样，这种分布式的操作模式对于开源软件社区的开发来说也是个巨大的恩赐，你不必再像以前那样做出补丁包，通过email方式发送出去，你只需要创建一个分支，向项目团队发送一个推请求。这能让你的代码保持最新，而且不会在传输过程中丢失。GitHub.com就是一个这样的优秀案例。

## 内容存储方式 ##
GIT把内容按元数据方式存储，而SVN是按文件。

什么是元数据（metadata, data about data）？可通过计算机中的文件来理解，如本文，存储为git.md，其真实数据就是展现在各位面前的内容。而在系统中，为方便文件系统管理，会为该文件生成一些元数据，使用stat命令查看git.md的元数据，包含文件名、inode、文件大小、文件实际占用的block数、相关时间参数等。

>元数据最大的好处是，它使信息的描述和分类可以实现格式化，从而为机器处理创造了可能。
>By startwithdp from csdn.net

```
sunnogo@a3e420:~/github/hexo/source/_posts/tools$ stat git.md
  File: ‘git.md’
  Size: 8491            Blocks: 24         IO Block: 4096   regular file
Device: 805h/2053d      Inode: 8654069     Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/ sunnogo)   Gid: ( 1000/ sunnogo)
Access: 2015-04-27 21:51:35.120007595 +0800
Modify: 2015-04-27 21:51:34.120007554 +0800
Change: 2015-04-27 21:51:34.164007556 +0800
 Birth: -

sunnogo@a3e420:~/github/hexo/source/_posts/tools$ stat -f git.md
  File: "git.md"
    ID: 212ada8747b1ce3b Namelen: 255     Type: ext2/ext3
Block size: 4096       Fundamental block size: 4096
Blocks: Total: 48028567   Free: 23741023   Available: 21295532
Inodes: Total: 12214272   Free: 11218615
```

这段话也还不理解：
>所有的资源控制系统都是把文件的元信息隐藏在一个类似.svn、.cvs等的文件夹里。如果你把.git目录的 体积大小跟.svn比较，你会发现它们差距很大。因为.git目录是处于你的机器上的一个克隆版的版本库，它拥有中心版本库上所有的东西，例如标签、分支、版本记录等。

## 分支 ##
SVN的分支看着相对简单，就是版本库中的另一个目录。

目前公司的SVN、GIT版本策略不是严格的branch/trunk/tags，分支太散太多，碎片太多，合代码相对难。有用过svn merge同步分支代码，但是还未用过直接整分支修订同步的命令，也不清楚有没有类似命令。

>你需要手工运行像这样的命令`svn propget svn:mergeinfo`，来确认代码是否被合并。所以，经常会发生有些分支被遗漏的情况。
>
>然而，处理GIT的分支却是相当的简单和有趣。你可以从同一个工作目录下快速的在几个分支间切换。你很容易发现未被合并的分支，你能简单而快捷的合并这些文件。

## 版本号 ##
SVN采用递增的全局版本号，GIT使用SHA-1来唯一标识一个代码快照。SVN的版本号很好理解，一次提交，本仓库的版本号就加1。

SVN从原始文件开始，每次记录有哪些文件作了更新，以及更新了哪些行的内容。GIT不一样，GIT不保存这些前后变化的差异数据，更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，它会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，GIT不会再次保存，而只对上次保存的快照作一链接。**问题：GIT快照是基于单一文件的，还是基于整个分支的？**

## 数据完整性 ##
GIT的SHA-1校验，能保证仓库的完整性。所有保存在GIT数据库中的东西都是使用此哈希值来作索引，而不是靠文件名。

[1]: http://www.bootcss.com/p/git-guide/index.html
[2]: /images/tools/git/trees.png
[3]: /images/tools/git/branches.png
[4]: http://www.oschina.net/news/12542/git-and-svn

