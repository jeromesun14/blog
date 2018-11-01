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

默认目标，即执行不带参数的 `make` 命令。只能配置一条规则，否则报错。例如 `.DEFAULT_GOAL := all`。

### .INTERMEDIATE

### .ONESHELL

### .SECONDEXPANSION

## 函数

### 使用 eval 在 makefile 规则中赋值

```
  $(eval IMAGE_TIME_STAMP=$(shell date +%Y%m%d%H%M%S))
  $(eval IMAGE_NAME_SIMPLE=$@)
  $(eval IMAGE_NAME=$(shell echo $(IMAGE_NAME_SIMPLE) | sed 's/.bin/.$(IMAGE_TIME_STAMP).bin/g'))
```

### foreach 遍历

### call 调用函数

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

### 在规则中使用 shell 函数

目前的写法还有点挫，不够优雅。

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
