title: python 学习笔记
date: 2018-06-13 09:02:24
toc: true
tags: [python]
categories: python
keywords: [python, list, dict, string to int, foreach]
description: python 学习笔记
---

## 运算符

* 逻辑运算符，没有 `|| &&`，换为 `and / or / not` 

## 全局变量 global
全局变量一般不建议使用，但是写小脚本的时候可能用到。
全局变量定义只需要在最外层定义即可，在函数内使用时，用 global 指定变量为外层定义的全局变量。

[样例](https://www.programiz.com/python-programming/global-keyword)：

```
c = 0 # global variable

def add():
    global c
    c = c + 2 # increment by 2
    print("Inside add():", c)

add()
print("In main:", c)
```

运行结果：

```
Inside add(): 2
In main: 2
```

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

赋值：

```
myDict = {}

myDict['test'] = "Value"

Python3:
myDict[b'test'] = "Value"
```

如果使用不当时，key 找不到，会出现 KeyError。此时通过 myDict.get(b'test', None)，判断返回值是否为 None，再进行进一步的操作。

### copy 赋值、浅复制和深复制的差异

http://www.runoob.com/w3cnote/python-understanding-dict-copy-shallow-or-deep.html

## 线程/锁

这里先只讨论 python 2.7 的。
线程和锁都隶属 [threading](https://docs.python.org/2/library/threading.html) 库。

threading 库包含几种 objects：

* Thread，创建、启动线程
* Lock，普通锁，不支持嵌套
* Rlock，嵌套锁
* Condition，条件锁
* Semaphore，信号量
* Timer，定时器
* Event

### Thread

### 在 with 语句中使用 locks, conditions, and semaphores

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

## 常见 error

### `TypeError: 'NoneType' object is not iterable in Python`

遍历时，对应的 list 或 dict 为空。详见 https://stackoverflow.com/questions/3887381/typeerror-nonetype-object-is-not-iterable-in-python。
可通过 `or []` 或 `or {}` 规避。

```
for i in data or []:
```

## 问题
### 想保密，如何发布 Python 组件？
### docker 中的 pyc 无法直接运行
### docker 中的 pyc 无法 import
