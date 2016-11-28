title: hexo 搭站备忘
date: 2016-03-16 09:48:18
toc: true
tags: [hexo]
categories: tools
keywords: [blog, hexo, npm, RSS, sitemap]
description: 本文描述 hexo 搭个人博客的记录，备忘。
---

## 安装 `hexo`

* 安装 `git`
* 安装 `nodejs`，debian / ubuntu 类的环境：`sudo apt-get install npm nodejs-legacy`
如果安装的是 `node` 包，则安装 hexo 时可能出现下文的 error 。

```
npm http GET https://registry.npm.taobao.org/inherits

> spawn-sync@1.0.15 postinstall /usr/local/lib/node_modules/hexo-cli/node_modules/hexo-util/node_modules/cross-spawn/node_modules/spawn-sync
> node postinstall

npm WARN This failure might be due to the use of legacy binary "node"
npm WARN For further explanations, please read
/usr/share/doc/nodejs/README.Debian

npm ERR! weird error 1
```

<!--more-->

* 安装 `hexo`，`sudo npm install -g hexo-cli`，如果想用淘宝的源：`npm install -g hexo-cli --registry=https://registry.npm.taobao.org`。实测官方源没有问题。
* 配置 npm 的默认源为淘宝源：`npm config set registry https://registry.npm.taobao.org/`


## 安装插件

* RSS: `npm install hexo-generator-feed`
* sitmap: `npm install hexo-generator-sitemap`
* deploy-git: `npm install hexo-deployer-git --save`，hexo 3.0 后必须。

## next 主题

next 主题 tags 页和 categories 页的设置与其他主题的不一样，会导致 tags 页和 categories 页没有内容。

以 tags 页为例：

```
---
title: tags
layout: tags  <-- 这个是之前的设置方式，不知道是不是 hexo 版本变化发生变化了。
type: "tags"  <-- 这个是当前 next 主题的配置方式。
date: 2016-03-18 00:32:32
---
```

