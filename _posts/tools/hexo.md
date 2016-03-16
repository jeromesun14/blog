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
* 安装 `hexo`，`npm install -g hexo-cli`，如果想用淘宝的源：`npm install -g hexo-cli --registry=https://registry.npm.taobao.org`。实测官方源没有问题。

<!--more-->

```
npm http GET https://registry.npm.taobao.org/inherits

> spawn-sync@1.0.15 postinstall /usr/local/lib/node_modules/hexo-cli/node_modules/hexo-util/node_modules/cross-spawn/node_modules/spawn-sync
> node postinstall

npm WARN This failure might be due to the use of legacy binary "node"
npm WARN For further explanations, please read
/usr/share/doc/nodejs/README.Debian

npm ERR! weird error 1
```

## 安装插件

* RSS: `npm install hexo-generator-feed`
* sitmap: `npm install hexo-generator-sitemap`

