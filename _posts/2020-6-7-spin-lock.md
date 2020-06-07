---
layout: post
title: Linux各种锁的实现
---


## 前言

本文是最近在学习Linux内核时的笔记，所用的源代码版本为2.6.0。

## 自旋锁如何实现

### 定义

```c
typedef struct {
	unsigned long magic;
	volatile unsigned long lock; // 1释放锁
	volatile unsigned int babble;
	const char *module;
	char *owner;
	int oline;
} spinlock_t;
```

### 实现


*如果不支持SMP，定义为空*

```c
#ifdef CONFIG_DEBUG_SPINLOCK
/*
 * If CONFIG_SMP is unset, declare the _raw_* definitions as nops
 */
#define spin_lock_init(lock)	do { (void)(lock); } while(0)
#define _raw_spin_lock(lock)	do { (void)(lock); } while(0)
#define spin_is_locked(lock)	((void)(lock), 0)
#define _raw_spin_trylock(lock)	((void)(lock), 1)
#define spin_unlock_wait(lock)	do { (void)(lock); } while(0)
#define _raw_spin_unlock(lock)	do { (void)(lock); } while(0)
#endif /* CONFIG_DEBUG_SPINLOCK */
```
### 加锁

在是否支持抢占，实现也是不同的

#### 支持抢占

```c
#define spin_lock(lock) \
do { \
	preempt_disable(); \ //禁止内核抢占
	if (unlikely(!_raw_spin_trylock(lock))) \
		__preempt_spin_lock(lock); \
} while (0)

```
#### 不支持抢占

```c
#define spin_lock(lock)	\
do { \
	preempt_disable(); \ //禁止内核抢占
	_raw_spin_lock(lock); \
} while(0)
```

先从简单的讲起，首先禁止了内核抢占，然后调用_raw_spin_lock 来加锁。

由于需要保证原子性，采用汇编实现，不同的机器上的具体实现也是不同的。如图。

![](/images/spin_lock_version.png)





这里展示了我最熟悉的x86_64上的实现。

```
static inline void _raw_spin_lock(spinlock_t *lock)
{
#ifdef CONFIG_DEBUG_SPINLOCK
	__label__ here;
here:
	if (lock->magic != SPINLOCK_MAGIC) {
printk("eip: %p\n", &&here);
		BUG();
	}
#endif
	__asm__ __volatile__(
		spin_lock_string
		:"=m" (lock->lock) : : "memory");
}
```

关键代码在于spin_lock_string这个宏

```
#define spin_lock_string \
	"\n1:\t" \
	"lock ; decb %0\n\t" \
	"js 2f\n" \
	LOCK_SECTION_START("") \
	"2:\t" \
	"rep;nop\n\t" \
	"cmpb $0,%0\n\t" \
	"jle 2b\n\t" \
	"jmp 1b\n" \
	LOCK_SECTION_END
```

原理：首先传入lock值，减一，lock指令用来保证原子操作

如果为负数，则进入循环，并且不断和0比较，直到大于0，才进入临界区

如果不为负数，直接进入临界区


接着讲讲__preempt_spin_lock实现

```c
void __preempt_spin_lock(spinlock_t *lock)
{
	if (preempt_count() > 1) {
		_raw_spin_lock(lock);
		return;
	}
	do {
		preempt_enable(); //允许内核抢占
		while (spin_is_locked(lock))
			cpu_relax();
		preempt_disable();
	} while (!_raw_spin_trylock(lock));
}
```
如果preempt_count值大于一(也就是禁止内核抢占)，那么用的还是之前的实现

先开启内核抢占，然后判断当前锁是否释放，如果不是，则循环等待释放。

关闭内核抢占。尝试加锁。



### 释放锁

```c
#define spin_unlock_string \
	"xchgb %b0, %1" \
		:"=q" (oldval), "=m" (lock->lock) \
		:"0" (oldval) : "memory"
```

代码很简单,直接将lock值赋值为1



***剩下的过两天再写***

## 读写锁如何实现
## 互斥锁如何实现
## 信号量如何实现



