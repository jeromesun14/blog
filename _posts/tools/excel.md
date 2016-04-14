
## 统计单元格数目

* 统计列满足条件的单元格个数，`COUNTIF` 函数
* 统计列满足多条件的单元格个数，`COUNTIFS` 函数

样例：

* 统计 "total" 标签所有 H 列，值等于 A12 的单元格个数

```
COUNTIF(total!H:H,A12)，
```

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

* 统计 "total" 标签所有 A 列值为 "*2015" （此处 `*` 为通配符）且 "total" 标签 I 列值为空的单元格个数

```
COUNTIFS(total!A:A,"*2015",total!I:I,"")
```
