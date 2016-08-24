title: GCC 问题记录
date: 2016-02-28 14:13:20
toc: true
tags: [gcc]
categories: programmer
keywords: [linux, gcc, compile, 编译]
description: Linux c gcc 使用问题记录。
---

## errors

### error: dereferencing pointer to incomplete type
原因：使用 `typedef struct {} xxx_t` 定义类型 `xxx_t`，定义变量时，使用 `struct xxx_t` ，应为 `xxx_t`。

```
typedef struct xxx_s {
    int a;
    int b;
} xxx_t;

int func(void)
{
   struct xxx_t st;
   
   st.a = 1; // <---- 这里会报错。
}
```