title: Linux命令行 - sed
date: 2017-12-5 20:03:59
tags: [Linux]
categories: shell
keywords: [linux, command, sed]
description: Linux 命令 sed 常用方法记录。
---

* -e，不写入文件
* -i，写入文件
* s，替换
  - s/xxx/yyy/g
  - s;xxx;yyy;

* 所有 subdir.mk 替换 "$(CC) $(BUILD_CFLAGS)" 为 “$(CC) $(BUILD_CFLAGS) -fPIC”，`find . -name "subdir.mk" -exec sed -i 's;$(CC) $(BUILD_CFLAGS);$(CC) $(BUILD_CFLAGS) -fPIC;' '{}' \;`
* 所有 .c 文件首行添加 “#include <asm-generic/io.h>”，`find . -name "*.c" -exec sed -i '1 i#include <asm-generic/io.h>' "{}" \;`
* 行首，行尾加字符，例如 # 号：`s/^/\#/g`，`s/$/\#/g`
* sed 替换匹配行的某个字符，例如还是行首加 # 号：`sed -i '/your_pattern/s/^/\#/g'`
* sed 匹配多个字符串，`sed -n '/hello\|world/p'` 或 `'/hello/p; /world/p'`
* 完全匹配字符串，`\<your_world\>`

## 问题汇总

### 记录一次二次展开的惨案

人生苦短，请用 Python...sb 了，这个搞了一个晚上，刚才才想起来，要用 `;` ...

```
jeromesun@kmcb0220:~/workshop/bash$ echo "abcdefghijk" | sed "s/abcd/fghi`pwd`/"
sed: -e expression #1, char 13: unknown option to `s'
jeromesun@km:~/workshop/bash$ echo "abcdefghijk" | sed "s;abcd;fghi`pwd`;"
fghi/home/jeromesun/workshop/bashefghijk
```
