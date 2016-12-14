
title: linux c atomic
date: 2016-12-15 01:03:24
toc: true
tags: [linux, c]
categories: c
keywords: [linux, c, atomic, 原子操作]
description: Linux c 原子操作。
---

原子操作也是同步的一种，信号量就是一个atomic_t。
kernel中，见asm-generic/atomic.h或asm-generic/bitops/atomic.h，据称不要
user space中gcc从4.1.1开始支持built-in atomic，但是有架构要求，详见gcc wiki，http://gcc.gnu.org/wiki/Atomic。
glibc atomic

下面这个atomic.h是从 http://golubenco.org/2007/06/14/atomic-operations/ down下来的，作者的愿意是替代kernel原有的atomic

```
/**
 * Atomic type.
 */
typedef struct {
    volatile int counter;
} atomic_t;

#define ATOMIC_INIT(i)  {(i)}

/**
 * Read atomic variable
 * @param v pointer of type atomic_t
 *
 * Atomically reads the value of @v.
 */
#define atomic_read(v) ((v)->counter)

/**
 * Set atomic variable
 * @param v pointer of type atomic_t
 * @param i required value
 */
#define atomic_set(v,i) (((v)->counter) = (i))

/**
 * Add to the atomic variable
 * @param i integer value to add
 * @param v pointer of type atomic_t
 */
static inline void atomic_add( int i, atomic_t *v )
{
    (void)__sync_add_and_fetch(&v->counter, i);
}

/**
 * Subtract the atomic variable
 * @param i integer value to subtract
 * @param v pointer of type atomic_t
 *
 * Atomically subtracts @i from @v.
 */
static inline void atomic_sub( int i, atomic_t *v )
{
    (void)__sync_sub_and_fetch(&v->counter, i);
}

/**
 * Subtract value from variable and test result
 * @param i integer value to subtract
 * @param v pointer of type atomic_t
 *
 * Atomically subtracts @i from @v and returns
 * true if the result is zero, or false for all
 * other cases.
 */
static inline int atomic_sub_and_test( int i, atomic_t *v )
{
    return !(__sync_sub_and_fetch(&v->counter, i));
}

/**
 * Increment atomic variable
 * @param v pointer of type atomic_t
 *
 * Atomically increments @v by 1.
 */
static inline void atomic_inc( atomic_t *v )
{
    (void)__sync_fetch_and_add(&v->counter, 1);
}

/**
 * @brief decrement atomic variable
 * @param v: pointer of type atomic_t
 *
 * Atomically decrements @v by 1.  Note that the guaranteed
 * useful range of an atomic_t is only 24 bits.
 */
static inline void atomic_dec( atomic_t *v )
{
    (void)__sync_fetch_and_sub(&v->counter, 1);
}

/**
 * @brief Decrement and test
 * @param v pointer of type atomic_t
 *
 * Atomically decrements @v by 1 and
 * returns true if the result is 0, or false for all other
 * cases.
 */
static inline int atomic_dec_and_test( atomic_t *v )
{
    return !(__sync_sub_and_fetch(&v->counter, 1));
}

/**
 * @brief Increment and test
 * @param v pointer of type atomic_t
 *
 * Atomically increments @v by 1
 * and returns true if the result is zero, or false for all
 * other cases.
 */
static inline int atomic_inc_and_test( atomic_t *v )
{
    return !(__sync_add_and_fetch(&v->counter, 1));
}

/**
 * @brief add and test if negative
 * @param v pointer of type atomic_t
 * @param i integer value to add
 *
 * Atomically adds @i to @v and returns true
 * if the result is negative, or false when
 * result is greater than or equal to zero.
 */
static inline int atomic_add_negative( int i, atomic_t *v )

{
    return (__sync_add_and_fetch(&v->counter, i) < 0);
}
```
