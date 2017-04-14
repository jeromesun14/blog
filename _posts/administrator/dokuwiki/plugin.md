title: 使用的 dokuwiki plugin
date: 2017-04-06 17:39:00
toc: true
tags: [dokuwiki]
categories: dokuwiki
keywords: [dokuwiki, plugin, 插件, markdown, discussion, 评论, 导航, 侧边栏, indexmenu]
description: dokuwiki 插件使用记录
---

# Markdown 语法支持

* 插件，[plugin:Markdowku](https://www.dokuwiki.org/plugin:Markdowku)，该插件可直接在 wiki 页上写 markdown 语法文本，不需要特殊处理。
* 用法，对比 [plugin:markdownextra](https://www.dokuwiki.org/plugin:markdownextra) 需要额外配置 Front Matter, Markdowku 可直接在 wiki 编辑页面使用 markdown 语法，不需要特殊的命名或语法块。

# 评论区

* 插件，[plugin:discussion](https://www.dokuwiki.org/plugin:discussion)
* 用法，`~~DISCUSSION~~` ，插入该语句到 wiki 中，即可在 wiki 内容后添加评论区。
* 配置，`管理`->`配置管理`->`Discussion`，比较有用的配置：
  + 订阅评论区
  + 通知版主有新评论
  + 允许未注册用户评论
  + 可通过 wiki 语法评论

# 导航
## indexmenu
默认在侧表栏显示2级目录，其他层级点击显示。

在 sidebar 页面输入：

```
{{indexmenu>..#2|navbar nocookie}}
```

## simplenavi
目录树好看，不过响应很慢。

# pdf 阅读器

* 插件，[plugin:pdfjs](https://www.dokuwiki.org/plugin:pdfjs)
* 用法，`{{pdfjs>:ns:document.pdf}}`

亦有 google docs 插件[gview](https://www.dokuwiki.org/plugin:gview)，不过用的 google docs 服务，天朝目前应该用不了。

# 显示最新修订

* 插件，[plugin:changes](https://www.dokuwiki.org/plugin:changes)

显示最近修订。

> This plugin allows you to embed a list of recent changes as a simple list into any page.

# 贡献统计

* 插件，[plugin:authorstats](https://www.dokuwiki.org/plugin:authorstats)

# 支持发出邮件

* 插件，[plugin:smtp](https://www.dokuwiki.org/plugin:smtp)

