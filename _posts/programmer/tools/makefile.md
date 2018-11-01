title: makefile 学习记录
date: 2016-04-15 20:03:24
toc: true
tags: [makefile]
categories: programmer
keywords: [makefile, compile, gcc, make]
description: makefile 使用备忘
---

## 特殊规则或变量

### .DEFAULT_GOAL

默认目标，即执行不带参数的 `make` 命令，例如 `.DEFAULT_GOAL := all`。只能配置一条规则，否则报错。

### .INTERMEDIATE

### .ONESHELL

Makefile 样例:

```Makefile

.ONESHELL:

test:
        @if [ ! -d abc ]; then
        @   mkdir abc
        @       echo "abc not found"
        @else
        @   echo "abc found"
        @fi
```

运行结果:

```
jeromesun@km:~/workshop/hello$ rmdir abc
jeromesun@km:~/workshop/hello$ make test
abc not found
jeromesun@km:~/workshop/hello$ make test 
abc found
```

删除 .ONESHELL，运行结果:

```
jeromesun@km:~/workshop/hello$ make test 
/bin/sh: 1: Syntax error: end of file unexpected
Makefile:10: recipe for target 'test' failed
make: *** [test] Error 2
```

### .SECONDEXPANSION

## 函数

### 自定义和使用函数

Makefile 样例如下，函数内的代码行不需要以 tab 开始。不以 @开头，在调用时也不会打印出 shell 语句。
通过 `define` 和 `endef` 围起来。

```makefile
.ONESHELL:

define func_test
    @echo "func_test parameters: 0:$0, 1:$1, 2:$2"
    @if [ ! -d abc ]; then
    @   mkdir abc
    @       echo "abc not found"
    @else
    @   echo "abc found"
    @fi
endef

test:
        @$(call func_test, one, two)
```

运行结果:

```
jeromesun@km:~/workshop/hello$ rmdir abc    
jeromesun@km:~/workshop/hello$ make test 
func_test parameters: 0:func_test, 1: one, 2: two
abc not found
jeromesun@km:~/workshop/hello$ make test 
func_test parameters: 0:func_test, 1: one, 2: two
abc found
```

### 使用 eval 在 makefile 规则中赋值

```
  $(eval IMAGE_TIME_STAMP=$(shell date +%Y%m%d%H%M%S))
  $(eval IMAGE_NAME_SIMPLE=$@)
  $(eval IMAGE_NAME=$(shell echo $(IMAGE_NAME_SIMPLE) | sed 's/.bin/.$(IMAGE_TIME_STAMP).bin/g'))
```

### foreach 遍历

### call 调用函数

**注意**: 

* Call 调用函数时，注意参数和逗号间不要有空格，否则参数分有带空格和没带空格之分
* 和 foreach 一起使用的时候，注意 call 的自定义函数最后一行要加 `;`

call 参数没空格样例:

```
jeromesun@km:~/workshop/hello.test$ cat mkfiles/func_test.mk 
.ONESHELL:

define func_test
    echo "func_test parameters: 0:$0, 1:$1, 2:$2"
    if [ ! -d abc ]; then
        mkdir abc
            echo "abc not found"
    else
       echo "abc found"
    fi
endef

jeromesun@km:~/workshop/hello.test$ cat Makefile 
.ONESHELL:

-include mkfiles/*.mk

test:
        @$(call func_test,one,two)
jeromesun@km:~/workshop/hello.test$ make test 
func_test parameters: 0:func_test, 1:one, 2:two
abc found
```

call 参数有空格样例:

```
jeromesun@km:~/workshop/hello.test$ cat Makefile 
.ONESHELL:

-include mkfiles/*.mk

test:
        @$(call func_test, one, two)
jeromesun@km:~/workshop/hello.test$ make 
func_test parameters: 0:func_test, 1: one, 2: two
abc found

```

## miscellaneous

### 判断 32 位还是 64 位

来自 [stackoverflow](http://stackoverflow.com/questions/4096173/how-do-i-create-a-single-makefile-for-both-32-and-64-bit) 的答案:

```
ARCH := $(shell getconf LONG_BIT)

CPP_FLAGS_32 := -D32_BIT ...  Some 32 specific compiler flags ...
CPP_FLAGS_64 := -D64_BIT

CPP_FLAGS := $(CPP_FLAGS_$(ARCH))  ... all the other flags ...
```

### 清除某个 CFLAGS 编译选项

来自 [stackoverflow](http://stackoverflow.com/questions/17316426/make-override-a-flag) 的答案：

```
CFLAGS := $(filter-out -Werror,$(CFLAGS))
```


### 在规则中等待输入

```
reset :
    @echo && echo -n "Warning! All local changes will be lost. Proceed? [y/N]: "
    @read ans && \
     if [ $$ans == y ]; then \
         git clean -xfdf; \
         git reset --hard; \
         git submodule foreach --recursive git clean -xfdf; \
         git submodule foreach --recursive git reset --hard; \
         git submodule update --init --recursive;\
     else \
         echo "Reset aborted"; \
     fi
```

### 【已淘汰】在规则中使用 shell 函数

这种写法很挫，不够优雅。用 .ONESHELL + 自定义 Makefile 函数替换。

```makefile
# $(1) path
# $(2) fetch cmd
# $(3) fetch url
# $(4) fetch branch
# $(5) fetch commit id
fetch_source =  \
    echo "Download Source 'path $1' 'cmd $2' 'url $3' 'branch $4' 'commit id $5'"; \
    if [ ! -d $(SOURCE_REAL)/$1 ]; then \
        mkdir -p $(SOURCE_REAL)/$1; \
        if [ "$2"i == "git"i ]; then \
            if [ -n "$4" ]; then \
                git clone -b $4 $3 $(SOURCE_REAL)/$1; \
            else \
                git clone $3 $(SOURCE_REAL)/$1; \
            fi; \
            pushd $(SOURCE_REAL)/$1 ; git checkout $5 ; git submodule update --init --recursive; popd; \
        elif [ "$2"i == "svn"i ]; then \
            svn chekcout $3 -r $5 $1; \
        elif [ "$2"i == "wget"i ]; then \
            wget $3 `basename $3` -P $(SOURCE_REAL)/$1; \
            pushd $(SOURCE_REAL)/$1; tar xvf `basename $3`; popd; \
        fi; \
        if [[ $1 =~ "platform/" ]]; then \
            ln -sf ../../$(SOURCE_REAL)/$1 $1; \
        else \
            ln -sf ../$(SOURCE_REAL)/$1 $1; \
        fi; \
    fi;

download :
    $(foreach target,$(SONIC_DOWNLOAD_SOURCE),$(call fetch_source,$($(target)_SRC_PATH),$($(target)_FETCH_CMD),$($(tar
get)_FETCH_URL),$($(target)_FETCH_BRANCH),$($(target)_FETCH_COMMITID)))
```

### 执行 shell 命令

例如 `PWD := $(shell pwd)`
