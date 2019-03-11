title: 一些常见的 git 命令备忘
date: 2019-01-31 09:43:33
toc: true
tags: [版本控制, git]
categories: tools
keywords: [git, command, version control]
description: 本文描述常见 git 命令备忘。
----------

本文首次编辑：2016-04-27。

* 查看 log，按 message 过滤、按 author 过滤、按日期过滤等。https://www.atlassian.com/git/tutorials/git-log。
* 灵活打 patch，https://www.jianshu.com/p/814fb6606734
* 第一次切换分支，git checkout -b xxx origin/xxx
* 后续切分支，git checkout xxx
* 删除分支，git branch -d xxx
* 查看当前分支，git branch -a | grep "*"
* git checkout到特定版本，git checkout upstream的前8位
* git clone / push 的时候不用密码：`git config --global credential.helper 'cache --timeout 7200'`，timeout 的单位为秒。
* 创建分支，`git checkout -b newbranch`
* 重命名分支，`git branch -m oldBranch newBranch`
* 提供本地分支到远程 repo（远程 repo 无此分支），`git push origin newBranch` or `git push -u origin newBranch`
* 强制推送本地修订，覆盖远程 repo。会覆盖此间别人的提交。`git push --force-with-lease`
* 打 patch 时，如何把 patch 中的 commit message 和 user 保持不变？`git am xxx.patch`，与 `git format-patch` 相对应

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
* git reset --hard commitid, git push, 回退到某个特定版本

* git remote -v，查看当前仓库的链接
* git status --untracked-files=no，git status 只查看跟踪的文件，同 git status -uno
* 回退某个文件的到特定版本，`git checkout c5f567 -- file1/to/restore file2/to/restore`，详见 [stackoverflow](git checkout c5f567 -- file1/to/restore file2/to/restore)

## 删除未被跟踪的任何文件和目录

**注意**：本操作会删除所有未加入 git 版本管理的文件。

在对应组件源码目录下，`git clean -f -d -X`，清除该目录中所有没有被跟踪的文件。


```
NAME
       git-clean - Remove untracked files from the working tree

SYNOPSIS
       git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] <path>...

DESCRIPTION
       Cleans the working tree by recursively removing files that are not under version control, starting from the current directory.

       Normally, only files unknown to Git are removed, but if the -x option is specified, ignored files are also removed. This can, for example, be useful to remove all build products.

       If any optional <path>... arguments are given, only those paths are affected.

OPTIONS
       -d
           Remove untracked directories in addition to untracked files. If an untracked directory is managed by a different Git repository, it is not removed by default. Use -f option twice if
           you really want to remove such a directory.

       -f, --force
           If the Git configuration variable clean.requireForce is not set to false, git clean will refuse to run unless given -f, -n or -i.

       -i, --interactive
           Show what would be done and clean files interactively. See “Interactive mode” for details.

       -n, --dry-run
           Don’t actually remove anything, just show what would be done.

       -q, --quiet
           Be quiet, only report errors, but not the files that are successfully removed.

       -e <pattern>, --exclude=<pattern>
           In addition to those found in .gitignore (per directory) and $GIT_DIR/info/exclude, also consider these patterns to be in the set of the ignore rules in effect.

       -x
           Don’t use the standard ignore rules read from .gitignore (per directory) and $GIT_DIR/info/exclude, but do still use the ignore rules given with -e options. This allows removing all
           untracked files, including build products. This can be used (possibly in conjunction with git reset) to create a pristine working directory to test a clean build.

       -X
           Remove only files ignored by Git. This may be useful to rebuild everything from scratch, but keep manually created files.
```

样例：

```
jerome@compile:~/sb/src/libttoo$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

        .gitignore
        debian/files
        debian/libttoo-dbg.debhelper.log
        debian/libttoo-dbg.substvars
        debian/libttoo-dbg/
        debian/libttoo-dev.debhelper.log
        debian/libttoo-dev.substvars
        debian/libttoo-dev/
        debian/libttoo.debhelper.log
        debian/libttoo.postinst.debhelper
        debian/libttoo.postrm.debhelper
        debian/libttoo.substvars
        debian/libttoo/
        debian/tmp/
        include/ttoo_version.patch
        output/x86-tt/bin/

nothing added to commit but untracked files present (use "git add" to track)
jerome@compile:~/sb/src/libttoo$ git clean -df
Removing .gitignore
Removing debian/files
Removing debian/libttoo-dbg.debhelper.log
Removing debian/libttoo-dbg.substvars
Removing debian/libttoo-dbg/
Removing debian/libttoo-dev.debhelper.log
Removing debian/libttoo-dev.substvars
Removing debian/libttoo-dev/
Removing debian/libttoo.debhelper.log
Removing debian/libttoo.postinst.debhelper
Removing debian/libttoo.postrm.debhelper
Removing debian/libttoo.substvars
Removing debian/libttoo/
Removing debian/tmp/
Removing include/ttoo_version.patch
Removing output/x86-tt/bin/
Removing output/x86-tt/objects/
jerome@compile:~/sb/src/libttoo$ git status
On branch master
nothing to commit, working directory clean
jerome@compile:~/sb/src/libttoo$
```

## git fork 一个仓库到本地后，同步远程仓库的修订

详见：

* [gitlab或github下fork后如何同步源的新更新内容？](https://www.zhihu.com/question/28676261)
* [同步一个 fork](https://gaohaoyang.github.io/2015/04/12/Syncing-a-fork/)
* [Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)

```
$ git clone http://192.168.199.32/sonic/libteam/ 
Cloning into 'libteam'...
remote: Counting objects: 4985, done.
remote: Compressing objects: 100% (1822/1822), done.
remote: Total 4985 (delta 3089), reused 4985 (delta 3089)
Receiving objects: 100% (4985/4985), 15.87 MiB | 0 bytes/s, done.
Resolving deltas: 100% (3089/3089), done.
Checking connectivity... done.
$ cd libteam/
$ git remote -v
origin  http://192.168.199.32/sonic/libteam/ (fetch)
origin  http://192.168.199.32/sonic/libteam/ (push)
$ git remote add upstream https://github.com/jpirko/libteam.git
$ git remote -v
origin  http://192.168.199.32/sonic/libteam/ (fetch)
origin  http://192.168.199.32/sonic/libteam/ (push)
upstream        https://github.com/jpirko/libteam.git (fetch)
upstream        https://github.com/jpirko/libteam.git (push)
$ git fetch upstream 
remote: Counting objects: 4, done.
remote: Total 4 (delta 3), reused 3 (delta 3), pack-reused 1
Unpacking objects: 100% (4/4), done.
From https://github.com/jpirko/libteam
 * [new branch]      gh-pages   -> upstream/gh-pages
 * [new branch]      master     -> upstream/master
$ git merge upstream/master 
Updating 3ede1b2..789591c
Fast-forward
 man/teamd.conf.5 | 7 ++++++-
 1 file changed, 6 insertions(+), 1 deletion(-)
$ git push origin master 
Username for 'http://192.168.199.32': your_name
Password for 'http://your_name@192.168.199.32': 
Counting objects: 4, done.
Delta compression using up to 24 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 511 bytes | 0 bytes/s, done.
Total 4 (delta 3), reused 0 (delta 0)
To http://192.168.199.32/sonic/libteam/
   3ede1b2..789591c  master -> master
```

如果远程“父”仓库的有很多 tags 和 branches，如何同步到本地远程 repo？

```
$ git remote add upstream https://github.com/opencomputeproject/SAI
$ git remote -v 
origin  http://192.168.199.32/sonic/SAI (fetch)
origin  http://192.168.199.32/sonic/SAI (push)
upstream        https://github.com/opencomputeproject/SAI (fetch)
upstream        https://github.com/opencomputeproject/SAI (push)
$ git fetch upstream
remote: Counting objects: 190, done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 190 (delta 119), reused 122 (delta 119), pack-reused 65
Receiving objects: 100% (190/190), 291.31 KiB | 250.00 KiB/s, done.
Resolving deltas: 100% (139/139), completed with 57 local objects.
From https://github.com/opencomputeproject/SAI
 * [new branch]      master     -> upstream/master
 * [new branch]      revert-648-vlanigmpcontrol -> upstream/revert-648-vlanigmpcontrol
 * [new branch]      v0.9.1     -> upstream/v0.9.1
 * [new branch]      v0.9.2     -> upstream/v0.9.2
 * [new branch]      v0.9.4     -> upstream/v0.9.4
 * [new branch]      v0.9.5     -> upstream/v0.9.5
 * [new branch]      v0.9.6     -> upstream/v0.9.6
 * [new branch]      v1.0       -> upstream/v1.0
 * [new branch]      v1.1       -> upstream/v1.1
 * [new branch]      v1.2       -> upstream/v1.2
 * [new tag]         v1.2.1     -> v1.2.1
 * [new tag]         v1.2.2     -> v1.2.2
 * [new tag]         v1.2.3     -> v1.2.3
 * [new tag]         v1.2.4     -> v1.2.4
 * [new tag]         v1.3.0     -> v1.3.0
$ git push --mirror 
Username for 'http://192.168.199.32': your_name
Password for 'http://your_name@192.168.199.32': 
Counting objects: 200, done.
Delta compression using up to 24 threads.
Compressing objects: 100% (93/93), done.
Writing objects: 100% (200/200), 351.49 KiB | 0 bytes/s, done.
Total 200 (delta 161), reused 145 (delta 107)
remote: Resolving deltas: 100% (161/161), completed with 60 local objects.
To http://192.168.199.32/sonic/SAI
 - [deleted]         revert-648-vlanigmpcontrol
 - [deleted]         riotmac
 - [deleted]         v0.9.1
 - [deleted]         v0.9.2
 - [deleted]         v0.9.4
 - [deleted]         v0.9.5
 - [deleted]         v0.9.6
 - [deleted]         v1.0
 - [deleted]         v1.1
 - [deleted]         v1.2
 * [new branch]      origin/HEAD -> origin/HEAD
 * [new branch]      origin/master -> origin/master
 * [new branch]      origin/revert-648-vlanigmpcontrol -> origin/revert-648-vlanigmpcontrol
 * [new branch]      origin/riotmac -> origin/riotmac
 * [new branch]      origin/v0.9.1 -> origin/v0.9.1
 * [new branch]      origin/v0.9.2 -> origin/v0.9.2
 * [new branch]      origin/v0.9.4 -> origin/v0.9.4
 * [new branch]      origin/v0.9.5 -> origin/v0.9.5
 * [new branch]      origin/v0.9.6 -> origin/v0.9.6
 * [new branch]      origin/v1.0 -> origin/v1.0
 * [new branch]      origin/v1.1 -> origin/v1.1
 * [new branch]      origin/v1.2 -> origin/v1.2
 * [new branch]      upstream/master -> upstream/master
 * [new branch]      upstream/revert-648-vlanigmpcontrol -> upstream/revert-648-vlanigmpcontrol
 * [new branch]      upstream/v0.9.1 -> upstream/v0.9.1
 * [new branch]      upstream/v0.9.2 -> upstream/v0.9.2
 * [new branch]      upstream/v0.9.4 -> upstream/v0.9.4
 * [new branch]      upstream/v0.9.5 -> upstream/v0.9.5
 * [new branch]      upstream/v0.9.6 -> upstream/v0.9.6
 * [new branch]      upstream/v1.0 -> upstream/v1.0
 * [new branch]      upstream/v1.1 -> upstream/v1.1
 * [new branch]      upstream/v1.2 -> upstream/v1.2
 * [new tag]         v1.2.1 -> v1.2.1
 * [new tag]         v1.2.2 -> v1.2.2
 * [new tag]         v1.2.3 -> v1.2.3
 * [new tag]         v1.2.4 -> v1.2.4
 * [new tag]         v1.3.0 -> v1.3.0
```

## 合并多次本地提交

* 不喜欢默认编译器 nano，先配置一下默认编辑器为 vim：`git config --global core.editor vim`
* git rebase -i HEAD~2，合并前两次提交

```
$ git log
commit 3d4729bc94a3a780afb2892c967ea23383933419
Author: your_name <your_name@your_company.com>
Date:   Mon Apr 2 20:30:16 2018 +0800

    [COMPILE] fix xxx issues

commit 19c2ee6f54ab1a860a54c38708024e69572a9358
Author: your_name <your_name@your_company.com>
Date:   Mon Apr 2 19:49:11 2018 +0800

    [Version Control] remove modules are not needed
$ git rebase -i HEAD~2
  GNU nano 2.5.3                                     File: /home/user/workshop/.git/rebase-merge/git-rebase-todo                                                                                 

pick 19c2ee6 [Version Control] remove modules are not needed
pick 3d4729b [COMPILE] fix xxx issues

# Rebase 8572f84..3d4729b onto 8572f84 (2 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

改要保留的提交 log 为 pick，把要合并的提交改为 squash。如上文改为 

```
pick 19c2ee6 [Version Control] remove modules are not needed
squash 3d4729b [COMPILE] fix xxx issues
```

保存退出后看 git log：

```
$ git log
commit 47c2b52a018680d314673fa11afd14d46a8841cf
Author: your_name <your_name@your_company.com>
Date:   Mon Apr 2 19:49:11 2018 +0800

    [Version Control] remove modules are not needed
    
    [COMPILE] fix xxx issues
```

## 处理 merge 冲突

* git pull 发现有冲突

```
$ git pull
Auto-merging .gitmodules
CONFLICT (content): Merge conflict in .gitmodules
Automatic merge failed; fix conflicts and then commit the result.
```

* 打开冲突的文件 .gitmodules

```
$ vi .gitmodules 
[submodule "src/repo"]
    path = src/repo
<<<<<<< HEAD
    url = http://192.168.250.250/repog/repo
=======
    url = http://192.168.250.250/repog/repo.git
>>>>>>> 19c2ee6f54ab1a860a54c38708024e69572a9358
```

* fix 对应的冲突
例如上文修订为：

```
[submodule "src/repo"]
    path = src/repo
    url = http://192.168.250.250/repog/repo
```

* 提交 fix 的结果

```
$ git add .gitmodules               
$ git commit -m "fix merge conflict"
[your_branch ee31e1a] fix merge conflict
```

* 提交代码

```
$ git push origin your_branch
Counting objects: 34, done.
Delta compression using up to 24 threads.
Compressing objects: 100% (33/33), done.
Writing objects: 100% (34/34), 3.49 KiB | 0 bytes/s, done.
Total 34 (delta 25), reused 0 (delta 0)
To http://192.168.250.250/repog/repox
   19c2ee6..ee31e1a  your_branch -> your_branch
```

## 如何不留 log 的 revert？（强制提交本地 repo 内容）

强制 revert 远程服务器的内容为本地 repo 内容。一般为 rebase 之后所需。

命令：`git push --force-with-lease`

但是如果不是洁癖，最好还是用 `git revert`，有记录地回退。

## 查看某个 commit id 属于哪个 tags 或 branch

* `git branch -r --contains commitid`
* `git tag --contains commitid`

## tag 相关操作

* `git tag -l` 列出全部 tag
* `git checkout <tag_name>`，detach 到某个 tag
* `git checkout tags/<tag_name> -b <branch_name>`，根据某个 tag 创建分支
* 查看某个 commmitid 是否属于某个 tag `git tag --contains commitid`

## 如何 merge 其他 repo 中的某次提交？

* 添加其他 repo，`git remote add upstream your_repo_url`
* 下载其他 repo 的代码：`git fetch upstream`
* 查看其他 repo 的 log：`git log upstream/master`
* 合并某次提交：`git cherry-pick your_repo_commit_id`

## git am 失败如何合代码

类似这个链接：https://blog.csdn.net/sunnylgz/article/details/7660638。

出现问题时：

* git apply --reject xxx.patch
* 到对应目录，应可以看到有对应的 .rej 文件，打开一看就可以看出是哪段代码合不进去。
* 手动处理冲突
* git add 对应文件
* git am --continue

## 查看本地分支与远程分支的差异

git status 的时候经常可以看到 `Your branch is ahead of 'origin/xxx' by 3 commits.`，有时候可能不知道这个 ahead / behind 的 commits 分别是什么。可通过 `git log origin/xxx..xxx` 查看，xxx 指分支名，以 master 分支为例，`git log origin/master..master`。

## git log 导入 excel

* [stackoverflow](https://stackoverflow.com/questions/39253307/export-git-log-into-an-excel-file)，`git log --pretty=format:%h,%an,%ae,%s > /path/to/file.csv`，然后导入 csv，以 "," 为分隔符隔开。前面 git log 输出格式为（hash [abbreviated], author name, author email, subject）。
* [类似 svn log --stop-on-copy 功能](https://stackoverflow.com/questions/22588648/git-svn-log-stop-on-copy-equivalent)，只取分支创建后的记录
  + 本地 git checkout 目标分支和目标分支的父分支。
  + `git log base_branch..target_branch --pretty=format:%h,%an,%ae,%s > your.csv`

## git add 提示 “error: insufficient permission for adding an object to repository database .git/objects”

* 原因：.git/objects 中的权限有问题
* 解决：见 https://stackoverflow.com/questions/6448242/git-push-error-insufficient-permission-for-adding-an-object-to-repository-datab, `sudo chown yourname:yourgroup -R .git/objects/`

```
$ git add openssh_7.9p1.orig.tar.gz.asc
error: insufficient permission for adding an object to repository database .git/objects
error: openssh_7.9p1.orig.tar.gz.asc: failed to insert into database
error: unable to index file openssh_7.9p1.orig.tar.gz.asc
fatal: adding files failed
$ ls -al .git/objects/
total 236
drwxrwxr-x 59 ubuntu ubuntu 4096 Mar  6 19:49 .
drwxrwxr-x  8 ubuntu ubuntu 4096 Mar  6 20:14 ..
drwxrwxr-x  2 ubuntu ubuntu 4096 Mar  6 19:49 01
drwxrwxr-x  2 ubuntu ubuntu 4096 Dec 25 21:46 03
drwxrwxr-x  2 ubuntu ubuntu 4096 Dec 25 21:46 06
drwxrwxr-x  2 ubuntu ubuntu 4096 Dec 28 16:12 0c
drwxrwxr-x  2 ubuntu ubuntu 4096 Jan 17 09:29 1c
drwxrwxr-x  2 ubuntu ubuntu 4096 Dec  4 09:44 20
drwxrwxr-x  2 ubuntu ubuntu 4096 Feb 19 09:39 29
drwxr-xr-x  2 root   root   4096 Jan 18 13:10 31
...
$ sudo chown ubuntu:ubuntu -R .git/objects/
[sudo] password for ubuntu: 
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        openssh_7.9p1.orig.tar.gz.asc

nothing added to commit but untracked files present (use "git add" to track)
$ git add openssh_7.9p1.orig.tar.gz.asc 
$
```
