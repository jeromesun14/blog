title: git中文编码问题
date: 2017-01-22 14:09:24
toc: true
tags: [git]
categories: tools
keywords: [git, 编码, utf-8, 中文]
description: git中文编码问题及其配置
---
来自 [解决git在Windows下的乱码问题](http://howiefh.github.io/2014/10/11/git-encoding/)。

解决前：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        "\344\270\264\346\227\266\346\226\207\346\241\243/"
        "\345\217\202\350\200\203\350\265\204\346\226\231/"
        "\346\213\223\345\261\225\345\237\271\350\256\255/"
        "\347\253\236\344\272\211\345\210\206\346\236\220/"
        "\350\247\243\345\206\263\346\226\271\346\241\210/"
        "\350\256\276\350\256\241\346\226\207\346\241\243/"
        "\351\241\271\347\233\256\347\256\241\347\220\206/"

nothing added to commit but untracked files present (use "git add" to track)
```

解决：

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ git config --global core.quotepath false
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ git config --global gui.encoding utf-8
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ git config --global i18n.commit.encoding utf-8
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ git config --global i18n.logoutputencoding utf-8
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ export LESSCHARSET=utf-8
```

解决后：
```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        临时文档/
        参考资料/
        拓展培训/
        竞争分析/
        解决方案/
        设计文档/
        项目管理/

nothing added to commit but untracked files present (use "git add" to track)
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ 
sunyongfeng@openswitch-OptiPlex-380:~/workshop/ne/ne-doc$ 
```
