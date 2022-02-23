---
title: "Go并发编程实战课笔记—Atomic"
date: 2022-01-29T02:28:06+08:00
draft: false
tags: ["Go","并发编程","note"]
categories: ["Go"]
slug: /golang/concurrent/0
---

# Go并发编程实战课笔记—Atomic

> 以下大多为鸟窝大佬的[Go 并发编程实战课](https://time.geekbang.org/column/intro/100061801) 中摘录的笔记
> 
> [代码repo](https://github.com/catwithtudou/golang_concurrent_examples/tree/master/atomic)


![image-20210222114638703](https://img.zhengyua.cn/20210222114638.png)

## 原子操作基础

在并发编程中，很多场景中使用并发原语较为复杂，而**原子操作可以更容易实现底层优化**。

因为**原子操作就是最小的粒子不可分割**，并不会像其他操作一样产生竞争的逻辑。由原子操作进行组合就能够实现比较复杂的指令或者操作。

CPU提供了基础的原子操作，但**不同的CPU架构甚至不同的版本提供的原子操作的指令是不同的，所以要用一种编程语言实现支持不同架构的原子操作是相当有难度的**。

在golang中将更底层的不同的架构下的实现封装成atomic包，提供了一个通用的原子操作API，包括了修改类型的原子操作（read-modify-write|RMW）和加载存储类型的原子操作（Load|Store）。

不同的架构中虽然代码一样，但产生的编译指令却是不同的，比如看起来貌似是一个原子操作但实际上却不是。

我们利用下面这段代码在不同的架构中通过`go tool compile`来分析其编译指令观察。

```go
package main


// 不同架构下相同的代码进行编译测试
// 观察其编译指令是否为原子操作

const x int64 = 1 + 1<<33

func main(){
	var i = x
	_ = i
}
```

- GOOS=linux GOARCH=386的架构编译

> 目前Darwin架构不支持386的架构

![image-20210222102441269](https://img.zhengyua.cn/20210222102446.png)

- GOOS=linux GOARCH=amd64的架构编译

![image-20210222103315098](https://img.zhengyua.cn/20210222103315.png)

可以看到在386架构是拆分成两条赋值语句来执行，amd64架构中是一条指令来执行。

## 应用场景

我们在某些场景可以使用Mutex等并发原语进行优化，虽然可以解决问题，但是这些**并发原语的逻辑比较复杂且对性能有一定的影响**，但我们**使用atomic的一些方法时可以实现更底层的部分优化**。

比如下面场景：

- **通过标志变量来判断某种状态**

考虑到读取和修改变量在并发时会产生竞争导致线程不安全，我们可以利用并发原语Mutex进行加锁保证互斥来解决问题，但此场景并不涉及到资源复杂的竞争逻辑，我们就可以使用atomic原子操作来操作该变量进行解决。

```go
func BenchmarkMutex(b *testing.B){
var k int64 = 1
l:=&sync.Mutex{}
var w sync.WaitGroup
w.Add(10)
b.ResetTimer()
for i:=0;i<10;i++{
go func() {
defer w.Done()
for j:=0;j<10000;j++{
l.Lock()
k = k + 1
l.Unlock()
}
}()
}
w.Wait()
}

func BenchmarkAtomic(b *testing.B){
var k int64 = 1
var w sync.WaitGroup
w.Add(10)
b.ResetTimer()
for i:=0;i<10;i++{
go func() {
defer w.Done()
for j:=0;j<10000;j++{
atomic.AddInt64(&k,1)
}
}()
}
w.Wait()
}
```

通过性能评测后可以得到如下结果：

![image-20210222110055469](https://img.zhengyua.cn/20210222110055.png)



有时候我们可以**使用atomic实现自己定义的基本并发原语**，在网上也能够搜集到相关的实现方法，且有些基本并发原语的底层就是基于通过atomic的方法实现的。

除此之外，**atomic原子操作也是实现lock-free数据结构的基石**，lock-free就是避免互斥锁来优化并发的性能。

## atomic提供的方法

因为golang**目前没有泛型的特性**，导致为了支持不同类型会分别为每种类型提供的相同的方法。

atomic**操作的对象是一个地址**，需要把可寻址的变量的地址作为参数传递给方法，而不是把变量的值传递给方法。

### Add

Add方法就是给第一个参数地址中的值增加一个delta值。

对于有符号的整数来说delta可以为负数，但对于无符号类型的没有提供减法操作，可以利用计算机补码的规则把减法变成加法。

```go
	atomic.AddUint32(&i,^uint32(0))
```

### CAS

```go
func CompareAndSwapInt32(addr *int32,old,new int32)(swapped bool)
```

即判断相等才替换。

### Swap

使用Swap方法替换后返回旧值。

### Load

取出addr地址中的值，即使在多处理器、多核、有CPU cache的情况下，此操作也能保证Load是一个原子操作。

### Store

把值存入到指定的addr地址中，即使在多处理器、多核、有CPU cache的情况喜爱，此操作也能保证Store是一个原子操作。

### Value

原子地存取对象类型，但只能存取。常常用在配置变更场景中，如下面代码实例。

```go
package main

import (
	"fmt"
	"go.uber.org/atomic"
	"math/rand"
	"sync"
	"time"
)

type Config struct{
	AddrName string
	Addr string
	Count int32
}

func loadNewConfig()Config{
	return Config{
		AddrName: "北京",
		Addr:     "0.0.0.1",
		Count:    rand.Int31(),
	}
}

func main(){
	var config atomic.Value
	config.Store(loadNewConfig())
	var cond = sync.NewCond(&sync.Mutex{})

	go func() {
		for{
			time.Sleep(time.Duration(5+rand.Int63n(5))*time.Second)
			config.Store(loadNewConfig())
			cond.Broadcast()
		}
	}()
	go func() {
		for{
			cond.L.Lock()
			cond.Wait()
			c := config.Load().(Config)
			fmt.Printf("new config:%+v\n",c)
			cond.L.Unlock()
		}
	}()

	select {

	}
}

```

## 第三方库的扩展

atomic的API较为简单，提供包一级的函数，函数调用起来较为麻烦。

第三方库的扩展中有些提供了面向对象的使用方式，如uber-go/atomic库。

## Lock-Free Queue

atomic常常用来实现Lock-Free的数据结构。下面就通过Lock-Free Queue代码实例来进行了解。

> 后面可以通过利用Mutex等并发原语来实现同样线程安全的数据结构来进行并发场景下性能的比较。

```go
package main

import (
	"sync/atomic"
	"unsafe"
)

type LKQueue struct{
	head unsafe.Pointer
	tail unsafe.Pointer
}

type node struct{
	value interface{}
	next unsafe.Pointer
}


func NewLKQueue()*LKQueue{
	n:=unsafe.Pointer(&node{})
	return &LKQueue{
		head: n,
		tail: n,
	}
}

func (q *LKQueue)Enqueue(v interface{}){
	n:=&node{value: v}
	for{
		tail:=load(&q.tail)
		next:=load(&tail.next)
		if tail == load(&q.tail){
			if next == nil{ //尾为空即没有数据入队
				if cas(&tail.next,next,n){ //增加到队尾
					cas(&q.tail,tail,n) //入队成功，移动尾巴指针
					return
				}
			}else{ //尾不为空则需要移动尾指针
				cas(&q.tail,tail,next)
			}
		}
	}
}

func (q *LKQueue)Dequeue()interface{}{
	for{
		head := load(&q.head)
		tail := load(&q.tail)
		next := load(&head.next)
		if head == load(&q.head){
			if head == tail{
				if next == nil{ //说明为空队列
					return nil
				}
				// 只是尾指针还没调整，尝试调整它指向下一个
				cas(&q.tail,tail,next)
			}else{
				//读取出队的数据
				v:=next.value
				if cas(&q.head,head,next){
					return v
				}
			}
		}
	}


}


func load(p *unsafe.Pointer)(n *node){
	return (*node)(atomic.LoadPointer(p))
}

func cas(p *unsafe.Pointer,old,new *node)(ok bool){
	return atomic.CompareAndSwapPointer(
		p,unsafe.Pointer(old),unsafe.Pointer(new))
}
```

## Atomic Vs &

我们可能会遇到一个疑问：对一个地址的赋值操作是否为原子操作？

如果是原子操作的话atomic包存在的意义是什么。

这里总结了一下DaveCheney谈论此问题是给出的解释：

在write的地址的对齐操作中有可能是需要处理器分成两个指令去处理，若执行一个指令，其他就会看到更新了一半的错误数据，这被称为撕裂写（torn write）。所以我们可以认为赋值操作是一个原子操作，这个“原子操作”可以认为是保证数据的完整性。

但在多处理多核的系统中，一个核对地址的更改在更新到主内存中之前，是在多级缓存中存放的，即会产生数据一致性的问题。为处理该类问题使用了内存屏障方式，简单来说就是进行操作时必须要等未完成的操作都被刷新到内存中，并且该操作还会让相关处理器的CPU缓存失效以此来拉取最新的值。

atomic包提供的方法会提供内存屏障的功能，所以atomic不仅可以保证赋值的数据完整性，还能保证多核中的数据可见性。但需要注意的是为了保证数据的一致性，atomic操作也是会降低性能的。

