title: moinmoin 配置过程记录
date: 2016-11-30 17:39:18
toc: true
tags: [moinmoin, administrator, wiki]
categories: administrator
keywords: [administrator, wiki, moinmoin, markdown, nginx]
description: 记录 moinmoin 配置过程。
---

## 主题配置
配置成 bootstrap 主题。

详见 [memodump](https://github.com/dossist/moinmoin-memodump/wiki/Installation) 安装过程。
screenshot：
![memodump](https://github.com/dossist/moinmoin-memodump/wiki/memodump.png)


## markdown 语法支持

安装使用 python markdown，`pip install markdown`。

```
root@ubuntu:~# pip install markdown
Downloading/unpacking markdown
  Downloading Markdown-2.6.7.zip (413kB): 413kB downloaded
  Running setup.py (path:/tmp/pip_build_root/markdown/setup.py) egg_info for package markdown
    
Installing collected packages: markdown
  Running setup.py install for markdown
    changing mode of build/scripts-2.7/markdown_py from 644 to 755
    Converting docs/siteindex.txt -> build/docs/siteindex.html
    Converting docs/release-2.0.1.txt -> build/docs/release-2.0.1.html
    Converting docs/release-2.0.2.txt -> build/docs/release-2.0.2.html
    Converting docs/install.txt -> build/docs/install.html
    Converting docs/index.txt -> build/docs/index.html
    Converting docs/release-2.4.txt -> build/docs/release-2.4.html
    Converting docs/release-2.2.1.txt -> build/docs/release-2.2.1.html
    Converting docs/authors.txt -> build/docs/authors.html
    Converting docs/release-2.1.0.txt -> build/docs/release-2.1.0.html
    Converting docs/change_log.txt -> build/docs/change_log.html
    Converting docs/release-2.3.txt -> build/docs/release-2.3.html
    Converting docs/cli.txt -> build/docs/cli.html
    Converting docs/release-2.6.txt -> build/docs/release-2.6.html
    Converting docs/release-2.2.0.txt -> build/docs/release-2.2.0.html
    Converting docs/release-2.0.txt -> build/docs/release-2.0.html
    Converting docs/release-2.5.txt -> build/docs/release-2.5.html
    Converting docs/release-2.1.1.txt -> build/docs/release-2.1.1.html
    Converting docs/reference.txt -> build/docs/reference.html
    Converting docs/test_suite.txt -> build/docs/test_suite.html
    Converting docs/extensions/definition_lists.txt -> build/docs/extensions/definition_lists.html
    Converting docs/extensions/footnotes.txt -> build/docs/extensions/footnotes.html
    Converting docs/extensions/admonition.txt -> build/docs/extensions/admonition.html
    Converting docs/extensions/abbreviations.txt -> build/docs/extensions/abbreviations.html
    Converting docs/extensions/index.txt -> build/docs/extensions/index.html
    Converting docs/extensions/fenced_code_blocks.txt -> build/docs/extensions/fenced_code_blocks.html
    Converting docs/extensions/sane_lists.txt -> build/docs/extensions/sane_lists.html
    Converting docs/extensions/tables.txt -> build/docs/extensions/tables.html
    Converting docs/extensions/api.txt -> build/docs/extensions/api.html
    Converting docs/extensions/meta_data.txt -> build/docs/extensions/meta_data.html
    Converting docs/extensions/code_hilite.txt -> build/docs/extensions/code_hilite.html
    Converting docs/extensions/smarty.txt -> build/docs/extensions/smarty.html
    Converting docs/extensions/wikilinks.txt -> build/docs/extensions/wikilinks.html
    Converting docs/extensions/header_id.txt -> build/docs/extensions/header_id.html
    Converting docs/extensions/extra.txt -> build/docs/extensions/extra.html
    Converting docs/extensions/nl2br.txt -> build/docs/extensions/nl2br.html
    Converting docs/extensions/toc.txt -> build/docs/extensions/toc.html
    Converting docs/extensions/smart_strong.txt -> build/docs/extensions/smart_strong.html
    Converting docs/extensions/attr_list.txt -> build/docs/extensions/attr_list.html
    
    changing mode of /usr/local/bin/markdown_py to 755
Successfully installed markdown
Cleaning up...
root@ubuntu:~# 
```

* moinmoin markdown 插件
详见 [ParserMarket/Markdown](https://moinmo.in/ParserMarket/Markdown)。
下载 `text_markdown.py` 到 `data/plugin/parser`。
修改 text_markdown.py，把 `output_html = markdown(self.raw)` 改为 `markdown(self.raw, extensions=['extra', 'abbr', 'attr_list', 'def_list', 'fenced_code', 'footnotes', 'tables', 'smart_strong', 'admonition', 'codehilite', 'headerid', 'meta', 'nl2br', 'sane_lists', 'smarty', 'toc', 'wikilinks', 'del_ins'])`，把你需要的 extra 往里加。

这里的 del_ins 是通过 ` pip install git+git://github.com/aleray/mdx_del_ins.git` 安装支持的。

```
"""
    MoinMoin - Parser for Markdown

    Syntax:

        To use in a code block:
    
            {{{{#!text_markdown
            <add markdown text here>
            }}}}

        To use for an entire page:

            #format text_markdown
            <add markdown text here>

    @copyright: 2009 by Jason Fruit (JasonFruit at g mail dot com)
    @license: GNU GPL, see http://www.gnu.org/licenses/gpl for details

"""


from markdown import markdown
from markdown.extensions import Extension

Dependencies = ['user']

class Parser:
    """
    A thin wrapper around a Python implementation
    (http://www.freewisdom.org/projects/python-markdown/) of John
    Gruber's Markdown (http://daringfireball.net/projects/markdown/)
    to make it suitable for use with MoinMoin.
    """
    def __init__(self, raw, request, **kw):
        self.raw = raw
        self.request = request
    def format(self, formatter):
        # output_html = markdown(self.raw)
	      output_html = markdown(self.raw, extensions=['extra', 'abbr', 'attr_list', 'def_list', 'fenced_code', 'footnotes', 'tables', 'smart_strong', 'admonition', 'codehilite', 'headerid', 'meta', 'nl2br', 'sane_lists', 'smarty', 'toc', 'wikilinks', 'del_ins'])
        try:
            self.request.write(formatter.rawHTML(output_html))
        except:
            self.request.write(formatter.escapedText(output_html))

```

## 中文语言支持配置

安装语言包，使用你配置的超级用户如 WikiAdmin 登陆。
访问wiki语言设置页面，根据自己的域名而修改，`http://localhost/LanguageSetup?action=language_setup`。
选择安装简体中文语言包，会看到提示：`附件'Simplified_Chinese--all_pages.zip'已安装`。

修改默认语言为中文
```
[root@syswiki moin]# vim /opt/syswiki/share/moin/wikiconfig.py 
…………略………… 
# The main wiki language, set the direction of the wiki pages
    language_default = 'zh'
…………略…………
```

重启 moinmoin 就可以看到页面换成中文。
