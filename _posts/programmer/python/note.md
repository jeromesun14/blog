title: python 学习笔记
date: 2018-06-13 09:02:24
toc: true
tags: [python]
categories: python
keywords: [python, list, dict, string to int, foreach]
description: python 学习笔记
---

## 运算符


详见 [菜鸟教程 python 运算符](http://www.runoob.com/python/python-operators.html)

- 算术运算符

| 运算符 | 描述                                            | 实例                                               |
| ------ | ----------------------------------------------- | -------------------------------------------------- |
| +      | 加 - 两个对象相加                               | a + b 输出结果 30                                  |
| -      | 减 - 得到负数或是一个数减去另一个数             | a - b 输出结果 -10                                 |
| *      | 乘 - 两个数相乘或是返回一个被重复若干次的字符串 | a * b 输出结果 200                                 |
| /      | 除 - x除以y                                     | b / a 输出结果 2                                   |
| %      | 取模 - 返回除法的余数                           | b % a 输出结果 0                                   |
| **     | 幂 - 返回x的y次幂                               | a**b 为10的20次方， 输出结果 100000000000000000000 |
| //     | 取整除 - 返回商的整数部分                       | 9//2 输出结果 4 , 9.0//2.0 输出结果 4.0            |

- 比较运算符，没有 `|| &&`，换为 `and / or / not` 

| 运算符 | 描述                                                         | 实例                                     |
| ------ | ------------------------------------------------------------ | ---------------------------------------- |
| ==     | 等于 - 比较对象是否相等                                      | (a == b) 返回 False。                    |
| !=     | 不等于 - 比较两个对象是否不相等                              | (a != b) 返回 true.                      |
| <>     | 不等于 - 比较两个对象是否不相等                              | (a <> b) 返回 true。这个运算符类似 != 。 |
| >      | 大于 - 返回x是否大于y                                        | (a > b) 返回 False。                     |
| <      | 小于 - 返回x是否小于y。所有比较运算符返回1表示真，返回0表示假。这分别与特殊的变量True和False等价。 | (a < b) 返回 true。                      |
| >=     | 大于等于	- 返回x是否大于等于y。                           | (a >= b) 返回 False。                    |
| <=     | 小于等于 -	返回x是否小于等于y。                           | (a <= b) 返回 true。                     |

- 赋值运算符

| 运算符 | 描述             | 实例                                  |
| ------ | ---------------- | ------------------------------------- |
| =      | 简单的赋值运算符 | c = a + b 将 a + b 的运算结果赋值为 c |
| +=     | 加法赋值运算符   | c += a 等效于 c = c + a               |
| -=     | 减法赋值运算符   | c -= a 等效于 c = c - a               |
| *=     | 乘法赋值运算符   | c *= a 等效于 c = c * a               |
| /=     | 除法赋值运算符   | c /= a 等效于 c = c / a               |
| %=     | 取模赋值运算符   | c %= a 等效于 c = c % a               |
| **=    | 幂赋值运算符     | c **= a 等效于 c = c ** a             |
| //=    | 取整除赋值运算符 | c //= a 等效于 c = c // a             |

- 位运算符

| 运算符 | 描述                                                         | 实例                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| &      | 按位与运算符：参与运算的两个值,如果两个相应位都为1,则该位的结果为1,否则为0 | (a & b) 输出结果 12 ，二进制解释： 0000 1100                 |
| \|     | 按位或运算符：只要对应的二个二进位有一个为1时，结果位就为1。 | (a \| b) 输出结果 61 ，二进制解释： 0011 1101                |
| ^      | 按位异或运算符：当两对应的二进位相异时，结果为1              | (a ^ b) 输出结果 49 ，二进制解释： 0011 0001                 |
| ~      | 按位取反运算符：对数据的每个二进制位取反,即把1变为0,把0变为1 。~x 类似于 -x-1 | (~a ) 输出结果 -61 ，二进制解释： 1100 0011，在一个有符号二进制数的补码形式。 |
| <<     | 左移动运算符：运算数的各二进位全部左移若干位，由 << 右边的数字指定了移动的位数，高位丢弃，低位补0。 | a << 2 输出结果 240 ，二进制解释： 1111 0000                 |
| >>     | 右移动运算符：把">>"左边的运算数的各二进位全部右移若干位，>> 右边的数字指定了移动的位数 | a >> 2 输出结果 15 ，二进制解释：                            |

- 逻辑运算符

| 运算符 | 逻辑表达式 | 描述                                                         | 实例                    |
| ------ | ---------- | ------------------------------------------------------------ | ----------------------- |
| and    | x and y    | 布尔"与" - 如果 x 为 False，x and y 返回 False，否则它返回 y 的计算值。 | (a and b) 返回 20。     |
| or     | x or y     | 布尔"或"	- 如果 x 是非 0，它返回 x 的值，否则它返回 y 的计算值。 | (a or b) 返回 10。      |
| not    | not x      | 布尔"非" - 如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。 | not(a and b) 返回 False |

- 成员运算符

除了以上的一些运算符之外，Python还支持成员运算符，测试实例中包含了一系列的成员，包括字符串，列表或元组。

| 运算符 | 描述                                                    | 实例                                            |
| ------ | ------------------------------------------------------- | ----------------------------------------------- |
| in     | 如果在指定的序列中找到值返回 True，否则返回 False。     | x 在 y 序列中 , 如果 x 在 y 序列中返回 True。   |
| not in | 如果在指定的序列中没有找到值返回 True，否则返回 False。 | x 不在 y 序列中 , 如果 x 不在 y 序列中返回 True |

- 身份运算符

| 运算符 | 描述                                        | 实例                                                         |
| ------ | ------------------------------------------- | ------------------------------------------------------------ |
| is     | is 是判断两个标识符是不是引用自一个对象     | **x is y**, 类似 **id(x) == id(y)** , 如果引用的是同一个对象则返回 True，否则返回 False |
| is not | is not 是判断两个标识符是不是引用自不同对象 | **x is not y** ， 类似 **id(a) != id(b)**。如果引用的不是同一个对象则返回结果 True，否则返回 False。 |


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

## 多线程处理

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

## 常用参数

### 延迟几秒

```python
import time

time.sleep(5)
```

### 字符串对比

`if a == 'abcdefg'`

## 常见 error

### `TypeError: 'NoneType' object is not iterable in Python`

遍历时，对应的 list 或 dict 为空。详见 https://stackoverflow.com/questions/3887381/typeerror-nonetype-object-is-not-iterable-in-python。
可通过 `or []` 或 `or {}` 规避。

```
for i in data or []:
```

### `RuntimeError: dictionary changed size during iteration`

字典 items() 和 iteritems() 的差异，都是遍历。
iteritems 使用时，不能变动字典的大小，即不能增删成员。

```
jeromesun@km:~/workshop$ python test_copy.py 
k 1 <type 'str'> v 1
k 3 <type 'str'> v 3
k 2 <type 'str'> v 2
k 4 <type 'str'> v 4
k 1 <type 'str'> v 1
Traceback (most recent call last):
  File "test_copy.py", line 10, in <module>
    for (k,v) in b.iteritems() or {}:
RuntimeError: dictionary changed size during iteration
jeromesun@km:~/workshop$ 
jeromesun@km:~/workshop$ cat test_copy.py    
import copy

a = {'1':1, '2':2, '3':3, '4':4}
for (k,v) in a.items() or {}:
    print "k {} {} v {}".format(k,type(k), v)
    del a[k]

a = {'1':1, '2':2, '3':3, '4':4}
b = copy.copy(a)
for (k,v) in b.iteritems() or {}:
    print "k {} {} v {}".format(k,type(k), v)
    #del a[k]
    b['5'] = 5
```

items 遍历的性能，在元素多的情况下，差很多：

```
jeromesun@km:~/workshop$ python test_copy.py 
len 1000 item 0.000249 a.k.a 4016064.25703 pps, iteritem 0.000116 a.k.a 8620689.65517 pps
len 10000 item 0.002774 a.k.a 3604902.66763 pps, iteritem 0.001042 a.k.a 9596928.98273 pps
len 100000 item 0.040658 a.k.a 2459540.55782 pps, iteritem 0.013075 a.k.a 7648183.55641 pps
len 1000000 item 0.724924 a.k.a 1379454.94976 pps, iteritem 0.176209 a.k.a 5675079.02548 pps
len 10000000 item 9.40743 a.k.a 1062989.57314 pps, iteritem 2.154777 a.k.a 4640851.4663 pps
jeromesun@km:~/workshop$ cat test_copy.py 
import copy
import datetime

def test_perf(size):
    a = {}
    for i in range(0, size):
        a[str(i)] = i

    item_start = datetime.datetime.now()
    for (k,v) in a.items() or {}:
        a[k] += 1
    item_end = datetime.datetime.now()

    iter_start = datetime.datetime.now()
    for (k,v) in a.iteritems() or {}:
        a[k] -= 1
    iter_end = datetime.datetime.now()

    titem = (item_end - item_start).total_seconds()
    titer = (iter_end - iter_start).total_seconds()
    print "len {} item {} a.k.a {} pps, iteritem {} a.k.a {} pps".\
        format(len(a), titem, size / titem, titer, size / titer)

test_perf(1000)
test_perf(10000)
test_perf(100000)
test_perf(1000000)
test_perf(10000000)
```

## 问题
### 多线程 python 程序运行时无法 ctrl + c 退出
TODO

### 想保密，如何发布 Python 组件？
### docker 中的 pyc 无法直接运行
### docker 中的 pyc 无法 import
