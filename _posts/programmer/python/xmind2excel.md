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

## freemind 格式分析

freemind 文件格式分析，xml 源码及 xmind mindmap 如下所示。

```
<?xml version="1.0" encoding="UTF-8"?>
<map version="0.8.1">
    <node TEXT="项目计划" MODIFIED="1492867311576" ID="0hai9mmspjp2s786pcir50olb1" CREATED="1492867311576">
        <node TEXT="相关信息" MODIFIED="1492867311576" ID="3a979etehnf0scej54cpf5ha4j" CREATED="1492867311576" POSITION="right">
            <icon BUILTIN="bookmark"/>
            <node TEXT="项目经理" MODIFIED="1492867311576" ID="734nep595s7klig55tfn46h3tr" CREATED="1492867311576">
                <hook NAME="accessories/plugins/NodeNote.properties">
                    <text>乔老爷子</text>
                </hook>
            </node>
            <node TEXT="团队成员" MODIFIED="1492867311576" ID="1omqdu3bt7s4he6ckksd7e3isl" CREATED="1492867311576"/>
            <node TEXT="项目介绍" MODIFIED="1492867311576" ID="7ftss79m00ap1okqk0eb0r49el" CREATED="1492867311576"/>
            <node TEXT="关键利益者" MODIFIED="1492867311576" ID="0p7adcg16k0gescfg56c6hpijo" CREATED="1492867311576"/>
            <node TEXT="背景介绍" MODIFIED="1492867311577" ID="4jjtr2fckvmmbb4hv05oe87b0u" CREATED="1492867311577">
                <node TEXT="example.xls" MODIFIED="1492867311579" ID="3b36u0f56pme6e577sbu69q04j" CREATED="1492867311577" LINK="images/3rjca88od8011qer034ihv0oan.xls"/>
	    </node>
        </node>
        <node TEXT="目标" MODIFIED="1492867311579" ID="2qouef1qf60nlp0gu85hvtpnle" CREATED="1492867311579" POSITION="right">
            <icon BUILTIN="bookmark"/>
        </node>
        <node TEXT="必要条件" MODIFIED="1492867311579" ID="3o4g7q484m4dn2prpn0t63fb9o" CREATED="1492867311579" POSITION="right">
            <icon BUILTIN="bookmark"/>
            <node TEXT="预算" MODIFIED="1492867311579" ID="1n8to0eqqbl7qpenj65m6rd5jq" CREATED="1492867311579"/>
            <node TEXT="人员" MODIFIED="1492867311579" ID="4886f7qdnnur0h33l4bjeuqjvr" CREATED="1492867311579"/>
            <node TEXT="资产" MODIFIED="1492867311579" ID="3hqhdaf3v3gr2bh7ttkq89ue06" CREATED="1492867311579"/>
            <node TEXT="2017.04.22 确认" MODIFIED="1492867311580" ID="3i0vobk9ovfplks8s78drjktki" CREATED="1492867311580"/>
	    </node>
        <node TEXT="<html><img src="images/57hi95enuhcl1c15525mg15r1o.png">" MODIFIED="1492867311580" ID="0ed31t5hvpa6hfevf213pollgh" CREATED="1492867311580" POSITION="right">
            <arrowlink ID="401c8v21ebt0buq4evf77ct7o0" STARTARROW="None" ENDARROW="Default" DESTINATION="6b0sc8ub8t8ji82o37j15j7pnt"/>
            <node TEXT="第一阶段" MODIFIED="1492867311582" ID="2uhjdmrfb533f2n1d8cd9rt1r2" CREATED="1492867311582">
                <node TEXT="高优先级" MODIFIED="1492867311582" ID="68on3d5udtcv2gb00qsl6drm5c" CREATED="1492867311582"/>
				<node TEXT="中等优先级" MODIFIED="1492867311582" ID="48ls3s61qj2vdstu7jsc51etb3" CREATED="1492867311582"/>
				<node TEXT="较低优先级" MODIFIED="1492867311582" ID="5c4rujcj281sorpd6l5fv6e712" CREATED="1492867311582"/>
			</node>
			<node TEXT="标志" MODIFIED="1492867311582" ID="2mnset6sop3p30ahgpgu0rasvp" CREATED="1492867311582"/>
			<node TEXT="第二阶段" MODIFIED="1492867311582" ID="4vvd2ij06hopllstsafc42qg7m" CREATED="1492867311582">
				<node TEXT="高优先级" MODIFIED="1492867311582" ID="5nhhs8090vqf9bsqqdf2be575d" CREATED="1492867311582"/>
				<node TEXT="中等优先级" MODIFIED="1492867311582" ID="2g0bvlskuq04gmetl0r5om4hsq" CREATED="1492867311582"/>
				<node TEXT="较低优先级" MODIFIED="1492867311582" ID="1shjdng4ufjvj2drsk2gc2ldrf" CREATED="1492867311582"/>
			</node>
			<node TEXT="标志" MODIFIED="1492867311582" ID="2km9h415v1c64th1ma9hpjovsa" CREATED="1492867311582"/>
			<node TEXT="第三阶段" MODIFIED="1492867311582" ID="4s4rf2536vmh7j1o7clh64be21" CREATED="1492867311582">
				<node TEXT="高优先级" MODIFIED="1492867311582" ID="542m4dbcci533bgs3b83vfu95m" CREATED="1492867311582"/>
				<node TEXT="中等优先级" MODIFIED="1492867311582" ID="1gj5005bf8s0q1brmv5lltbab8" CREATED="1492867311582"/>
				<node TEXT="较低优先级" MODIFIED="1492867311582" ID="0r65aga724a778tionvopubipd" CREATED="1492867311582"/>
			</node>
			<node TEXT="标志" MODIFIED="1492867311582" ID="3ncr651qia6e4cv2vposgusuhu" CREATED="1492867311582"/>
		</node>
		<node TEXT="<html><img src="images/701at6fa712kqn6r5nb6ri83ud.png">" MODIFIED="1492867311582" ID="6b0sc8ub8t8ji82o37j15j7pnt" CREATED="1492867311582" POSITION="left">
			<node TEXT="已完成的任务" MODIFIED="1492867311611" ID="071u3nqfdiic6sau5f7fdg3ctr" CREATED="1492867311611"/>
			<node TEXT="取消的任务" MODIFIED="1492867311611" ID="6tj8kg88oumtel0ki8lu0jjncp" CREATED="1492867311611"/>
			<node TEXT="被延迟的任务" MODIFIED="1492867311611" ID="2nl2npf0774uu7374uhsv640ad" CREATED="1492867311611"/>
			<node TEXT="暂停的任务" MODIFIED="1492867311611" ID="1ab39j3upshgvgsja654na18lr" CREATED="1492867311611"/>
			<node TEXT="进行中的任务" MODIFIED="1492867311611" ID="3duu8bv3jol9htep4brrhj27tp" CREATED="1492867311611"/>
		</node>
		<node TEXT="风险" MODIFIED="1492867311611" ID="3r7div183nssfcspufbed5gsut" CREATED="1492867311611" POSITION="left">
			<node TEXT="风险1" MODIFIED="1492867311611" ID="79qem186f8retr8h2m6p94fh9s" CREATED="1492867311611">
				<node TEXT="描述" MODIFIED="1492867311611" ID="747p6vp33fp925p1rsc5tpdvhh" CREATED="1492867311611"/>
				<node TEXT="可能的影响" MODIFIED="1492867311611" ID="3e5tj66vk777rmmuck8sd9hf4s" CREATED="1492867311611"/>
				<node TEXT="严重程度" MODIFIED="1492867311611" ID="2v578nqjtjmc0us7qpiq9mfg67" CREATED="1492867311611"/>
				<node TEXT="可能性" MODIFIED="1492867311611" ID="3caf7q4vrsjbh9tutrslkv1ceg" CREATED="1492867311611"/>
				<node TEXT="事前检测出来的可能性" MODIFIED="1492867311611" ID="6f3rjgd0pbahphscurp4s0b6b6" CREATED="1492867311611"/>
				<node TEXT="相应的缓解方法" MODIFIED="1492867311611" ID="178k163mtpbfn817dp7nk72n5t" CREATED="1492867311611"/>
				<node TEXT="推荐的解决方案" MODIFIED="1492867311611" ID="4s839q2p2hikk0qs8e60fnropd" CREATED="1492867311611"/>
			</node>
			<node TEXT="风险2" MODIFIED="1492867311611" ID="26k9binqncjakkb6kq2jrooibj" CREATED="1492867311611">
				<node TEXT="描述" MODIFIED="1492867311611" ID="6ftav360fer4ei9vud0rou4mn0" CREATED="1492867311611"/>
				<node TEXT="可能的影响" MODIFIED="1492867311611" ID="2mp59ni55sicoqccj7ioka737g" CREATED="1492867311611"/>
				<node TEXT="严重程度" MODIFIED="1492867311611" ID="4mshvotfm0c3t8qn2sr34pk4as" CREATED="1492867311611"/>
				<node TEXT="可能性" MODIFIED="1492867311611" ID="27ibg9mm2lurv62m84sr17jk1a" CREATED="1492867311611"/>
				<node TEXT="事前检测出来的可能性" MODIFIED="1492867311611" ID="6443iv9kjlsn7rglj3tqsn65vd" CREATED="1492867311611"/>
				<node TEXT="相应的缓解方法" MODIFIED="1492867311611" ID="7dirapfhic24srerve84vnod1u" CREATED="1492867311611"/>
				<node TEXT="推荐的解决方案" MODIFIED="1492867311611" ID="2e1qlidincv7jtg3aeuh2p5m3j" CREATED="1492867311611"/>
			</node>
		</node>
	</node>
</map>
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

* 问题：xmind bug，图片所在节点的文字不会导出到 freemind 中。

```
<node TEXT="<html><img src="images/57hi95enuhcl1c15525mg15r1o.png">" MODIFIED="1492867311580" ID="0ed31t5hvpa6hfevf213pollgh" CREATED="1492867311580" POSITION="right">
```

综上，需归递解析出 freemind 的每个 node 标签，并分析是否为叶子节点，根据是否叶子节点判断excel row 是否加一，将 node 标签的 TEXT 属性值写入 excel 第 row 行，第 level 列。

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

使用到的 python 语法：

* if 语句，通过 `is not None` 判断非空。

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

## 第三版 —— 带参数解析

* python 参数解析内嵌库 [argparse](https://docs.python.org/2/library/argparse.html)
* python 参数解析方法汇总，[Python中的命令行解析工具介绍](http://lingxiankong.github.io/blog/2014/01/14/command-line-parser/)
* python argparse 使用详解，[argparse - 命令行选项与参数解析（译）](http://blog.xiayf.cn/2013/03/30/argparse/)

本文只需要带个参数，一是输入文件，二是输出文件，其中输入文件为必选选项，输出文件为可选。

源码：

```
#!/usr/bin/python

import xml.etree.ElementTree as ET
import xlwt
import argparse

g_row = 0

def do_write_excel(text, row, col):
    #print text + " row %d + col %d"%(row, col)
    ws.write(row, col, text) 

def perf_func(elem, func, level = 0):
    global g_row
    func(elem, g_row, level)

    for child in list(elem):
        name = child.get('TEXT')

        perf_func(child, func, level + 1)
        if child.find('node') is None and name is not None:
            g_row = g_row + 1

def write_excel(elem, row, level):
    name = elem.get('TEXT')
    if name is not None:
        do_write_excel(name, row, level)

parser = argparse.ArgumentParser()

parser.add_argument('-i', '--input-file', type=str, dest='inputfile', required=True)
parser.add_argument('-o', '--output-file', type=str, dest='outputfile', default='freemind2excel.xls', help='Default outputfile is freemind2excel.xls')
args = parser.parse_args()

if args.inputfile is None:
    parser.print_help()
    exit()

root = ET.parse(args.inputfile)
map_version = root.getroot()
first_node = map_version.find('node')

wb = xlwt.Workbook()
ws = wb.add_sheet('freemind2excel')

perf_func(first_node, write_excel)

wb.save(args.outputfile)
```

运行结果：

```
sunnogo@DESKTOP-VM2TU8I:~/workshop/xmind2excel$ ./xmind2excel.py
usage: xmind2excel.py [-h] -i INPUTFILE [-o OUTPUTFILE]
xmind2excel.py: error: argument -i/--input-file is required
sunnogo@DESKTOP-VM2TU8I:~/workshop/xmind2excel$ ./xmind2excel.py -i project_plan.mm
sunnogo@DESKTOP-VM2TU8I:~/workshop/xmind2excel$ ls *.xls
freemind2excel.xls
sunnogo@DESKTOP-VM2TU8I:~/workshop/xmind2excel$ ./xmind2excel.py -i project_plan.mm -o test.xls
sunnogo@DESKTOP-VM2TU8I:~/workshop/xmind2excel$ ls *.xls
freemind2excel.xls  test.xls
```
