title: 一些常见的 git 命令备忘
date: 2016-04-27 20:33:18
toc: true
tags: [版本控制, git]
categories: tools
keywords: [git, command, version control]
description: 本文描述常见 git 命令备忘。


* stage

Q：多一个 stage 有什么优点？
A：暂存空间。git add 后的文件放到 stage 区，再修改同样的文件，其修订不会体现在此次 commit，除非再 git add 一次。

* git format-patch，已提交版本生成 patch
* git am -3 --ignore-space-change，只能使用 format-patch 的 patch
* git config core.editor vim
* git checkout file，可以让 file 从 staged 状态返回 untracked 状态
  + git checkout -- file
* git reset HEAD file，让文件从 staged 返回 modified 状态
* git clean -f -d，删除未被跟踪的任何文件/目录
* .gitignore，忽略不需要跟踪的文件
* git revert commitid，撤消某个修订并自动 commit
* git reset --soft，回退修订
* 基于某个 commitid 创建新分支 name
  + git reset --hard commitid
  + git branch name
* git merge，不推荐使用。可通过 git format 打 patch，优势是可以看到同步的每个修订。
* git cherry-pick commitid，单独同步某次提交
* git show commit
* git commit --amend，修改最后一次提交的 log
* git rebase，不推荐新手使用。
