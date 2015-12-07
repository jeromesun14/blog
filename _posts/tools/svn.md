title: SVN基础用法
date: 2015-01-08 22:48:18
toc: true
tags: [版本控制, svn]
categories: tools
keywords: [svn, command, version control]
description: 本文描述SVN的基础使用方法。
---

svn - Subversion command line client tool

这里讲SVN命令行客户端工具如何使用，不涉及SVN服务器如何部署。

# 创建新分支 #

```
svn copy src_url [-r version] dst_url [-m "message"] [--username your_name --password your_password]
```

如果不带`-r version`，则默认使用src_url的最新版本。version为数字，版本号。  
建议所有含提交功能的命令都要带`-m message`，这样才能通过log直接明了地看出这个版本提交了什么修订。

**注意**，user 需同时拥有src_url和dst_url的权限，才可创建新分支成功，否则会提示仅限问题。

# 下载代码 #
```
svn checkout url -r version
```

从服务器仓库下载代码到本地，成为本地工作副本。

<!--more-->

# 升级到新版本 #
```
svn update -r version
```

# 查看工作副本状态 #
```
svn status
```

第一列表示文件的状态：

* ` `，没有修订
* `A`，添加
* `C`，冲突，需要解决冲突状态，才能正常提交代码。
* `D`，删除
* `I`，忽略
* `M`，有修改
* `?`，没有版本控制，在工作副本添加文件或目录之后，需要使用`svn add your_path`才能加该文件加到版本控制。
* `!`，文件丢失，如果不是使用`svn delete`删除文件或目录，会产生此状态。

# 添加新文件或目录到版本控制 #
```
svn add file1 file2 ...
svn add dir ...  
```

使用Linux命令或在窗口下添加文件或目录后，需要使用本命令，才能将添加的文件或目录加入版本控制，svn提交时才能将该文件或目录提交到服务器。  

# 删除文件或目录 #
```
svn delete your_path
```

如果仅仅是手动使用rm命令或窗口下删除工作副本内的文件或目录，该删除并不会记录svn的状态。可能会导致提交代码时，遗漏了删除文件或目录。因此建议删除svn工作副本内的文件或目录时，使用本命令进行操作。

# 重命名文件或目录 #
```
svn move src dst
```

**问题**：`svn move`重命名文件之后，再用`svn diff`打patch会发现只能打进删除文件的补丁，没有新增文件的内容，目前还不清楚`svn move`要如何打patch。例如：

```
sunnogo@a3e420:~/src/test$ svn status
sunnogo@a3e420:~/src/test$ svn mv my.spec tmp.spec
A         tmp.spec
D         my.spec
sunnogo@a3e420:~/src/test$ svn status
D       my.spec
        > moved to tmp.spec
A  +    tmp.spec
        > moved from my.spec
sunnogo@a3e420:~/src/test$ svn diff
Index: my.spec
===================================================================
--- my.spec     (revision 11706)
+++ my.spec     (working copy)
@@ -1,27 +0,0 @@
-#ʹ����ȷ�������滻�ļ��а����ַ�'X'�ĵط�
-
-Summary: my packages
- 此处省略N行。
-%attr(755,root,root)
-/*
sunnogo@a3e420:~/src/test$ 

```


# 查看工作副本信息 #
```
svn info
```

能够查看到本工作副本的url、版本等信息。

# 生成patch #
```
svn diff [file_list]
```

将工作副本的修订以patch的形式输出，常使用`svn diff > your_patch.patch`输出patch。

# 打某个版本的patch #
```
svn diff -r ver1:ver2 [file_list]
```

查看某两个版本中的修订，如果ver1大于ver2，则所输出的diff是回退代码的patch；如果ver2大于ver1，则所输出的patch是合并代码的patch。

# 应用补丁 #
```
svn patch your_patch.patch
```

# 提交代码 #
```
svn commit [-m message] [file_list]
```

如果没有带文件列表，则把工作副本的所有修订都提交，如果有带文件列表，则只提交文件列表中对应文件的修订。

# 合并代码 #
```
svn merge -r ver1:ver2 src_url working_copy_path
```

可将任意版本的任意修订合并到工作副本中。如果ver1小于ver2，表示合并src_url分支ver1到ver2的修订到本地工作副本；如果ver1大于ver2，表示**回退修订**。

另外也可以操作服务器仓库，把working_copy_path直接换成目的分支的url即可，但是这种做法比较危险，不建议新手直接使用。

# 回退工作副本的修订 #
```
svn revert file1 file2 ...
svn revert -R dir
```

# 查看log #
```
svn log [OPTIONS] [FILE_LIST]
```

会默认输出所有的log，不实用，需要使用参数过滤才能得到我们想要的内容。  
默认只查看工作副本及以前版本的log。

常用参数

* `-l n`，只输出n个log信息；
* `-v`，显示修订的文件，默认不显示；
* `-r ver`，显示特定版本的log；
* `-r ver1:ver2`，显示版本ver1到ver2之间所有提交的log；
* `-r {2013-01-01}:{2013-01-11}`，显示日期间所有提交的log，-r选项的版本号和日期可以混用；
* `--diff`，除显示log信息外，还直接输出修订的内容；
* `--search`，根据当前输出的log信息，按关键字过滤log，该关键字可以匹配输出信息的任意字符串，比如可匹配到提交者、提交的log信息，如果带`-v`选项还可以匹配到修订的文件等。
* `--stop-on-copy`，只显示当前分支的修订的log，不会回溯源基线分支修订的log。比如分支branch基于tags分支版本100创建，此时branch分支的svn log会默认显示tags分支版本100以前所有修订的log，而如果带上本选项，则只会显示branch分支自己修订的log。


