title: Excel 使用记录
date: 2016-04-12 22:23:00
toc: true
tags: [Excel]
categories: tools
keywords: [office, excel]
description: Excel 使用备忘，主要是函数相关
---

## 统计单元格数目

* 统计列满足条件的单元格个数，`COUNTIF` 函数
* 统计列满足多条件的单元格个数，`COUNTIFS` 函数

样例：

* 统计 "total" 标签所有 H 列，值等于 A12 的单元格个数

```
COUNTIF(total!H:H,A12)，
```

<!--more-->

* 统计 "total" 标签所有 H 列，值为 "abc" 的单元格个数

```
COUNTIF(total!H:H,"abc")
```

* 统计 "total" 标签所有 H 列，非空单元格个数

```
COUNTIF(total!H:H,"<>")
```

* 统计 "total" 标签所有 H 列，空单元格个数

```
COUNTIF(total!H:H,"")
```

* 统计 "total" 标签所有 A 列值为 `*2015` （此处 `*` 为通配符）且 "total" 标签 I 列值为空的单元格个数

```
COUNTIFS(total!A:A,"*2015",total!I:I,"")
```


## 删除重复数据
`数据` 选项卡 -> `数据工具` 选项组中的 `删除重复项`，按提示操作。

## 按日期算周数

类似 `=CEILING(("2011-3-14"-"2011-2-10")/7,1)`，可以把日期改成按日期格式填写内容的单元格

## 饼图数字百分比到小数点后面

右击饼图中的数字，选择`设置数据标签格式`，选择第二个大类 `数据` -> `类别` -> `百分比`，`小数点位数` 为你所需要的。
