title: xmind 转换成 excel
date: 2017-04-20 21:39:00
toc: true
tags: [python]
categories: python
keywords: [python, xmind, excel, export, convert, 导出, 转换]
description: 实现 xmind pro 才有的功能 —— 转换 xmind 为 excel。
---

## 思路

思路直接使用 [百码山庄](http://seejs.me) 的文章 [【原创】XMind免费到Excel的全过程](http://seejs.me/2016/04/30/xmind2excel/)。该文使用 nodejs / php 实现本功能，本文使用 python 实现。

1. 转换 xmind 文件为 freemind `.mm` 后缀文件。
2. freemind 文件实为 xml 格式，通过解析 xml 文件，按格式导出为 excel。

## 第一版 —— xml 递归解析
### 参考代码
参考代码，[Python - How to determine hierarchy level of parsed XML elements?](http://stackoverflow.com/questions/15748528/python-how-to-determine-hierarchy-level-of-parsed-xml-elements) 中 pradyunsg 的回答，通过递归的获取子节点的形式达到获取 xml 等级的目的。

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

### 适配到 freemind 的解析
实现获取属性 'TEXT'，参考 [xml.etree.ElementTree — The ElementTree XML API](https://docs.python.org/2/library/xml.etree.elementtree.html#finding-interesting-elements)。

freemind 文件格式分析，xml 源码及 xmind mindmap 如下所示。

```

```

* 根节点

```
<map version="0.8.1">`
```

* mindmap 节点

```
<node TEXT="项目计划" MODIFIED="1492867311576" ID="0hai9mmspjp2s786pcir50olb1" CREATED="1492867311576">
```

* 联系节点，即 xmind 中的箭头，ctrl + l

```
<arrowlink ID="221s6rn1ft46fc6207057chd1k" STARTINCLINATION="43;180" STARTARROW="None" ENDARROW="Default" DESTINATION="61q31b1bilu0mpim374q7hn048"/>
```

* 备注节点，node 节点的子节点。

```
<node TEXT="项目经理" MODIFIED="1492867311576" ID="734nep595s7klig55tfn46h3tr" CREATED="1492867311576">
  <hook NAME="accessories/plugins/NodeNote.properties">
    <text>乔老爷子</text>
  </hook>
</node>
```

* 概要节点，同 mindmap 节点

```
<node TEXT="2017.04.22 确认" MODIFIED="1492867311580" ID="3i0vobk9ovfplks8s78drjktki" CREATED="1492867311580"/>
```

* 特殊标签 —— 图标

```
<icon BUILTIN="bookmark"/>
```



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

## 第二版 —— 导出为 excel

缺点：使用全局变量

```
#!/usr/bin/python

import xml.etree.ElementTree as ET
import xlwt

g_row = 0

def do_write_excel(text, row, col):
    print text + " row %d + col %d"%(row, col)
    ws.write(row, col, text) 

def perf_func(elem, func, level = 0):
    global g_row
    func(elem, g_row, level)

    # elem maybe not a node tag
    # if elem.tag is 'node':

    for child in list(elem):
        name = child.get('TEXT')

        perf_func(child, func, level + 1)
        if child.find('node') is None and name is not None:
            g_row = g_row + 1

def write_excel(elem, row, level):
    name = elem.get('TEXT')
    if name is not None:
        do_write_excel(name, row, level)

    '''
    if elem.find('node') is None and name is not None:
        print "leaf" + '-' * level + name
    elif name is not None:
        print '-' * level + name
    '''

root = ET.parse('sde.xml')
map_version = root.getroot()
first_node = map_version.find('node')

wb = xlwt.Workbook()
ws = wb.add_sheet('freemind2excel')

perf_func(first_node, write_excel)

wb.save('freemind2excel.xls')
```

使用到的 Python 语法：

* 字符串判空，`if a == b`，注意 is 是用于判断是否同一个对象。
* list 判空，`if not a`。
* 全局变量，在函数前定义，在函数内要定义为 global

* etree.ElementTree
  + 获取 elementTree 树对象，parse(source)
  + 获取 element 对象，getroot
  + 获取 tag 的值，elem.tag
  + 获取首个标签为 node 的 element，elem.find('node')
  + 获取属性 TEXT 的值，elem.get('TEXT')

* xlwt
  + 实例化，xlwt.workbook()
  + 添加 sheet，add_sheet
  + 保存 xls 文件，save
  + 写数据，write

下一步计划：

* 使用类封装
* 参数解析，带参数

