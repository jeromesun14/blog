title: python 学习笔记
date: 2018-06-13 09:02:24
toc: true
tags: [python]
categories: python
keywords: [python, list, dict, string to int, foreach]
description: python 学习笔记
---

## 数据结构
### list
#### 遍历列表

见 [Python 列表(List) 的三种遍历(序号和值)方法](https://www.cnblogs.com/pizitai/p/6398276.html)，这里贴一下原作者 **痞子泰** 的代码备忘，

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
if __name__ == '__main__':
    list = ['html', 'js', 'css', 'python']

    # 方法1
    print '遍历列表方法1：'
    for i in list:
        print ("序号：%s   值：%s" % (list.index(i) + 1, i))

    print '\n遍历列表方法2：'
    # 方法2
    for i in range(len(list)):
        print ("序号：%s   值：%s" % (i + 1, list[i]))

    # 方法3
    print '\n遍历列表方法3：'
    for i, val in enumerate(list):
        print ("序号：%s   值：%s" % (i + 1, val))

    # 方法3
    print '\n遍历列表方法3 （设置遍历开始初始位置，只改变了起始序号）：'
    for i, val in enumerate(list, 2):
        print ("序号：%s   值：%s" % (i + 1, val))
```

#### 列表比较

python 2.7 直接使用 cmp() 函数比较，0 表示相等。

#### 列表排序

详见 [菜鸟教程](http://www.runoob.com/python/att-list-sort.html)，

```
list.sort(cmp=None, key=None, reverse=False)
```

key 表示要比较的元素，贴一下样例代码：

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 获取列表的第二个元素
def takeSecond(elem):
    return elem[1]
 
# 列表
random = [(2, 2), (3, 4), (4, 1), (1, 3)]
 
# 指定第二个元素排序
random.sort(key=takeSecond)
 
# 输出类别
print '排序列表：', random
```

输出结果：

```
排序列表：[(4, 1), (2, 2), (1, 3), (3, 4)]
```


### dict

## print

两种格式化方法：

* 使用 format，`print '{name} {test}'format(name='yyy', test='zzz')`
* 原生，`print '%s %s'%('yyy', 'zzz')`

## syslog

* syslog
  + format: `syslog.syslog('%%Hello: %s'%('your_name'))`
* logging


## 条件语句

```
if condition:
   statement
elif condition:
   statement
else
   statement
```
