title: 一些常见的 git 命令备忘
date: 2016-04-27 20:33:18
toc: true
tags: [版本控制, git]
categories: tools
keywords: [git, command, version control]
description: 本文描述常见 git 命令备忘。
----------

* 第一次切换分支，git checkout -b xxx origin/xxx
* 后续切分支，git checkout xxx
* 查看当前分支，git branch -a | grep "*"
* git checkout到特定版本，git checkout upstream的前8位

* stage

Q：多一个 stage 有什么优点？
A：暂存空间。git add 后的文件放到 stage 区，再修改同样的文件，其修订不会体现在此次 commit，除非再 git add 一次。

* git format-patch，已提交版本生成 patch
  + `git format-patch -<n> <SHA1>`，输出从 SHA1 开始的 n 个提交 patch，会输出 10 个 patch
  + `git format-patch -10 HEAD --stdout > 0001-last-10-commits.patch`，输出最新前 10 次修订为一个 patch
* `git log -p -1 <commit>`，查看某次提交的具体修订
* `git diff sha1 sha2 > mypatch.diff`，输出 sha1 到 sha2 的所有修订，如果 sha1 比 sha2 早，则是修订 patch，如果晚，则是回退 patch。
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

* git remote -v，查看当前仓库的链接

## 删除未被跟踪的任何文件和目录

* `git clean -f -d`
