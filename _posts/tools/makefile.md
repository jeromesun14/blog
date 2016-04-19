makefile 学习记录
=================

## miscellaneous

### 判断 32 位还是 64 位

来自 [stackoverflow](http://stackoverflow.com/questions/4096173/how-do-i-create-a-single-makefile-for-both-32-and-64-bit) 的答案:

```
ARCH := $(shell getconf LONG_BIT)

CPP_FLAGS_32 := -D32_BIT ...  Some 32 specific compiler flags ...
CPP_FLAGS_64 := -D64_BIT

CPP_FLAGS := $(CPP_FLAGS_$(ARCH))  ... all the other flags ...
```
