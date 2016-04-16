Linux 链接库使用记录
====================

本文记录 Linux 静态、动态链接库的一些常用使用方法。

## 查看符号

* `nm` 命令，`nm -A libxxx.so 2>/dev/null | grep "your_symbol"`
