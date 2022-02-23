---
title: "Go并发编程实战课笔记—Etcd"
date: 2022-01-29T02:28:06+08:00
draft: false
tags: ["Go","并发编程","note"]
categories: ["Go"]
slug: /golang/concurrent/4
---


# Go并发编程实战课笔记—Etcd

> 以下为鸟窝大佬的[Go 并发编程实战课](https://time.geekbang.org/column/intro/100061801) 中摘录的笔记
> 
> [代码repo](https://github.com/catwithtudou/golang_concurrent_examples/tree/master/etcd)



![image-20210315103855322](https://img.zhengyua.cn/20210315103900.png)



ETCD 提供了较多的分布式并发原语，比如分布式互斥锁、分布式读写锁、Leader选举。

## Leader 选举

主从架构的服务节点分为主（Leader、Master）和从（Follower、Slave）两种角色。

主节点通常执行写操作，从节点通常执行读操作，若读写都在主节点而从节点提供备份则主从架构就会退化成主备模式架构。

选主机制就是选择一个节点作为主节点，保证主节点的唯一性从而保证数据的一致性。

### 选举

有以下和选主相关的提供的方法：

```go
// 把一个节点选举为主节点且会设置一个值
// 这是一个阻塞方法，在调用它的时候会被阻塞，直到满足下面的三个条件之一，才会取消阻塞。成功当选为主；此方法返回错误；ctx 被取消
func (e *Election) Campaign(ctx context.Context,val string)error

// 重新设置 Leader 的值，但是不会重新选主，这个方法会返回新值设置成功或者失败的信息
func (e *Election) Proclaim(ctx context.Context, val string) error

// 开始新一次选举。这个方法会返回新的选举成功或者失败的信息
func (e *Election) Resign(ctx context.Context) (err error)
```

### 查询

有以下和查询相关的提供的方法：

```go
// 查询当前 Leader 的方法 Leader，如果当前还没有 Leader，就返回一个错误
// 可以使用这个方法来查询主节点信息。
func (e *Election) Leader(ctx context.Context) (*v3.GetResponse, error)

// 每次主节点的变动都会生成一个新的版本号，你还可以查询版本号信息（Rev 方法），了解主节点变动情况
func (e *Election) Rev() int64
```

### 监控

有以下和监控相关的提供的方法：

```go
// 监控主节点的变化
// 它会返回一个 chan，显示主节点的变动信息
// 需要注意的是它不会返回主节点的全部历史变动信息，而是只返回最近的一条变动信息以及之后的变动信息
func (e *Election) Observe(ctx context.Context) <-chan v3.GetResponse
```

## 互斥锁

主要关注在分布在不同机器中的不同进程内的 goroutine，如何利用分布式互斥锁来保护共享资源。

**在同一时刻，分布式互斥锁在所有节点中只允许其中的一个节点持有锁。**

### Locker

类似于标准库中的 sync.Locker 接口，提供了 Lock/Unlock 的机制：

```go
func NewLocker(s *Session,pfx string) sync.Locker
```

### Mutex

Mutex 并没有实现 sync.Locker 接口，它的 Lock/Unlock 方法需要提供一个 context.Context 实例做参数，可以得知在请求锁的时候，你可以设置超时时间，或者主动取消请求。

## 读写锁

ETCD 提供的读写锁，可以在分布式环境中的不同的节点使用。

提供的方法是和功能与标准库中的读写锁一致，分别提供了 RLock/RUnlock、Lock/Unlock方法。

> 如果持有互斥锁或者读写锁的节点意外宕机了，它持有的锁会被释放。这里我们可以关注对应的 session 相关处理。
>
> etcd 提供的读写锁中的读和写按照实现来说是写锁应该比读锁优先级高。