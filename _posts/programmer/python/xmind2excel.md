title: xmind 转换成 excel
date: 2017-04-20 21:39:00
toc: true
tags: [python]
categories: python
keywords: [python, xmind, excel, export, convert, 导出, 转换]
description: 实现 xmind pro 才有的功能 —— 转换 xmind 为 excel。
---

首先，转换 xmind 文件为 freemind .mm 格式文件。
freemind 文件为 xml 格式，可通过解析 xml 文件，再导为 excel。

## 解析 xml 文件

参考代码：http://stackoverflow.com/questions/15748528/python-how-to-determine-hierarchy-level-of-parsed-xml-elements 中 pradyunsg 的回答。

通过递归的获取子节点的形式达到获取 xml 等级的目的。

原代码：

```python
import xml.etree.ElementTree as ET

def perf_func(elem, func, level=0):
    func(elem,level)
    for child in elem.getchildren():
        perf_func(child, func, level+1)

def print_level(elem,level):
    print '-'*level+elem.tag

root = ET.parse('XML_file.xml')
perf_func(root.getroot(), print_level)
```

修改后的代码：（实现获取属性 'TEXT'，参考 https://docs.python.org/2/library/xml.etree.elementtree.html#finding-interesting-elements）

```python
#!/usr/bin/python

import xml.etree.ElementTree as ET

def perf_func(elem, func, level=0):
    func(elem,level)
    for child in elem.getchildren():
        perf_func(child, func, level+1)

def print_level(elem,level):
    name = elem.get('TEXT')
    if name is not None:
        print '-'*level + name

root = ET.parse('test.xml')
perf_func(root.getroot(), print_level)
```

## 导出 excel

TBD
