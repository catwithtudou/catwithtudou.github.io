---
title: "Go并发编程实战课笔记—Cond"
date: 2022-01-29T02:28:06+08:00
draft: false
tags: ["Go","并发编程","note"]
categories: ["Go"]
slug: /golang/concurrent/2
---


# Go并发编程实战课笔记—Cond


> 以下大多为鸟窝大佬的[Go 并发编程实战课](https://time.geekbang.org/column/intro/100061801) 中摘录的笔记
>
> [代码repo](https://github.com/catwithtudou/golang_concurrent_examples/tree/master/cond)


![image-20210107002455090](https://img.zhengyua.cn/img/20210107002455.png)

## Go标准库的Cond

Go 标准库提供 Cond 原语的目的是，为等待 / 通知场景下的并发问题提供支持。

Cond通常应用于等待某个条件的一组goroutine，等条件变为true的时候，其中一个goroutine或者所有的goroutine都会被唤醒执行。

- 开发实践中使用到Cond场景比较少，且Cond场景一般也能用Channel方式实现，所以更多人会选择使用Channel。

## Cond基本用法

标准库中的Cond并发原语初始化的时候需要关联一个Locker接口的实例，一般使用Mutex或者RWMutex。

```go
type Cond
	func NeWCond(l Locker) *Cond
	func (c *Cond) Broadcast() //允许调用者Caller唤醒所有等待此Cond的goroutine即清空Cond等待队列中所有的goroutine
	func (c *Cond) Signal() //允许调用者Caller按等待队列顺序唤醒一个等待此Cond的goroutine
	func (c *Cond) Wait() //会把调用者Caller放入Cond的等待队列中并阻塞，直到被Signal或者Broadcast方法从等待队列中唤醒。
```

> Go 实现的 sync.Cond 的方法名是 Wait、Signal 和 Broadcast，这是计算机科学中条件变量的通用方法名。

## Cond的实现原理

```go
type Cond struct {
	noCopy noCopy
	// 当观察或者修改等待条件的时候需要加锁
	L Locker
	// 等待队列
	notify notifyList
    // 辅助结构，在运行时检查Cond是否被复制使用
	checker copyChecker
}
func NewCond(l Locker) *Cond {
	return &Cond{L: l}
}
func (c *Cond) Wait() {
	c.checker.check()
	// 增加到等待队列中
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	// 阻塞休眠直到被唤醒
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}
func (c *Cond) Signal() {
	c.checker.check()
	runtime_notifyListNotifyOne(&c.notify)
}
func (c *Cond) Broadcast() {
	c.checker.check()
	runtime_notifyListNotifyAll(&c.notify）
}
```

## 使用注意

- **调用 Wait 的时候没有加锁**。
- 只调用了一次Wait，没有检查等待条件是否满足，结果条件没满足，程序就继续执行了。

