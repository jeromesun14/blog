title: linux c printf 相关记录
date: 2016-12-15 01:03:24
toc: true
tags: [linux, c]
categories: c
keywords: [linux, c, container_of, get structure head, 结构体, 成员, 头指针]
description: linux c 通过结构体成员获取结构体头指针。
---

结构体成员取结构体头指针

list_entry->container_of->offsetof

344 /**
345  * list_entry - get the struct for this entry
346  * @ptr:        the &struct list_head pointer.
347  * @type:       the type of the struct this is embedded in.
348  * @member:     the name of the list_struct within the struct.
349  */
350 #define list_entry(ptr, type, member) \
351         container_of(ptr, type, member)

684 /**
685  * container_of - cast a member of a structure out to the containing structure
686  * @ptr:        the pointer to the member.
687  * @type:       the type of the container struct this is embedded in.
688  * @member:     the name of the member within the struct.
689  *
690  */
691 #define container_of(ptr, type, member) ({                      \
692         const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
693         (type *)( (char *)__mptr - offsetof(type,member) );})

Linux/include/linux/stddef.h

  1 #ifndef _LINUX_STDDEF_H
  2 #define _LINUX_STDDEF_H
  3
  4 #include <linux/compiler.h>
  5
  6 #ifdef __KERNEL__
  7
  8 #undef NULL
  9 #define NULL ((void *)0)
 10
 11 enum {
 12         false   = 0,
 13         true    = 1
 14 };
 15
 16 #undef offsetof
 17 #ifdef __compiler_offsetof
 18 #define offsetof(TYPE,MEMBER) __compiler_offsetof(TYPE,MEMBER)
 19 #else
 20 #define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
 21 #endif
 22 #endif /* __KERNEL__ */
 23
 24 #endif
 25

__compiler_offsetof(TYPE,MEMBER)就和gcc的原子操作实现一样，也是编译器built-in
