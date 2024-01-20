---
title: "初识🐣Rust🦀️"
date: 2023-11-13T00:00:00+08:00
lastmod: 2024-01-10T00:00:00+08:00
draft: false
tags: ["rust"]
categories: ["rust"]
slug: /rust/2
---

> 此文档内容为飞书文档复制过来作为搜索，存在内容格式不兼容情况，建议看原[飞书文档](https://jih9axn4gg.feishu.cn/wiki/KDGFwGkbziEj7ukNlUjc1omtncc?from=from_copylink)

## 背景

最近在学习 Rust，体验了大家所说的 Rust 陡峭的学习曲线（经历反复拿起和放下的横跳体验），在前期接触时确实是非常打击学习的自信心，其核心原因就是 Rust 走了一条与其他编程语言方向完全不同的路，导致了若你有其他语言的经验，你就能十分明显的感受到一种可能会无法适应的“割裂感”。

作为 Rust 的初学者和目前主要使用语言是 Golang 的服务端同学，通过此篇来分享本人在接触 Rust 过程中所遇到的一些有趣的设计思想（不会过于深入语言的具体细节），其中部分也会通过 Go 语言进行比较来直接体验差异。期望能做到：

- **减轻同学想要学习 Rust 的心智负担（提前体验“割裂感”）**
- **初步了解到 Rust 的特性和设计思想，拓展没有接触过该语言的同学的编程视角**
- **初步了解到 Rust 语法和和与其他语言存在差异的部分等**

## 语言简介

### Rust 的发展历程

暂时无法在飞书文档外展示此内容

Rust 最早是 Mozilla 雇员 Graydon Hoare 的个人项目，在 2006 年首次出现。当时 Mozilla 开发 firefox 的 Servo 引擎想要保证安全的同时保持高性能，于是瞄准 Rust，在 2009 年开始得到 Mozilla 研究院的赞助，并在 2010 年作为官方项目对外公布，2010 到 2011 年间实现自举。自此之后，Rust 在重构与崩溃之间反复横跳，比如早期 Rust 实际上是有 GC 的，但在 在 1.0 前夕做出决定去掉 GC。最终在 2015 年发布 1.0 版本，确定了 Rust 的高安全性和高性能的特性，同时也意味着稳定性的保证。在开发期间 Rust 也建立一个强大且活跃的社区，形成一整套完善稳定的项目贡献机制。

与其它语言相比，Rust 的更新迭代较为频繁（得益于精心设计的发布流程及开发者团队的严格管理）：

- 每 6 周发布一个迭代版本
- 2-3 年发布一个新的大版本，如 Rust 2018 edition，Rust 2021 edition
    - **2015～2018 年**：主要解决“生产力”问题，使工具链、文档、编译器等更加智能且对开发者更加友好
    - **2018～2021 年**：完善异步生态，此时 Rust 也开始适用于写业务了，毕竟同步写业务会遇到 C10k 问题
    - **2021～2024 年**：Rust 提出“扩展授权”的规划，其原因是 Rust 的目标是“empower everyone to build reliable and efficient software”，而官方团队也关注到了大伙的痛点即“Rust 难学”问题，所以在完善 Rust 的基础特性后，开始着重关注在易用性和项目落地上

而目前**在开源方面**，大量的项目正在被 Rust 重写（即“一切能用 Rust 重写的项目都将或者正在用 Rust 重写”），在 UI 层开发、基础设施层、数据库、搜索引擎、系统开发、操作系统、区块链等多个方面的开源项目整体都在爆发性增长。同时**在使用现状方面**，很多世界知名企业也都关注着 Rust 语言发展，开始用 Rust 语言来解决问题和构建应用，而通过占据连续七年 StackOverflow 最受开发者喜爱语言榜的榜首也能看出，对个人开发者也是极其受欢迎的。

### Rust 的特点

1. **为什么 Rust 如此受欢迎呢？**

这就需要回答一个老生常谈的问题即“为什么又来一个新的编程语言？”而简单来说就是目前还没有这样一种语言，**无 GC 且无需手动内存管理、性能高、工程性强、语言级安全性以及能同时得到工程派和学院派认可的语言**，而 Rust 就是这样的语言，可以说解决了一些现有编程语言的内存管理问题痛点、兼顾工程化和高性能和从源头上提升代码质量等问题。比如在 Rust 之前：

- 想要追求极致性能但难以保证安全，代表语言是 C/C++
    - **若 Rust 代码能够成功编译，则能保证内存就是安全的**
- 想要开发效率但难以保证性能和安全，代表语言是 Go/Java
    - **简单来说仅经过了简单优化的 Rust 版本，远超手动深度调优过的 Go 版本**

除此之外 Rust 还提供良好的协作开发体验及相关工具链，可以说 Rust 是**「性能｜安全｜协作」的集语言大成者。**

1. **当然没有任何一门语言是完美的，Rust 也有让人没有那么舒服的地方**

- 学习曲线陡峭，**概念和特性较复杂和不好理解**，具体开发时的错误提示能让你反复怀疑之前学的
    - 比如既没有 Java 和 Go 的逃逸分析和 GC，也没有 C++ 的手动内存分配，需要理解 Rust 特殊的“所有权”
- 虽然语言还在快速发展中，但**生态还是不如其他成熟的编程语言那样完善**
- 因语言特性（需要保证自引用类型的内存安全），会带来**地狱难度之“尝试用 Rust 写个链表”**
- 与目前热门编程语言有着较多**设计的细节差异**，如`;`、`::`、`'a`等

### Rust 适合被改造的服务

- 请求量较大、资源占用较高
- GC占比较高、CPU受限较多、延时波动
- 代码业务逻辑改动较少（比如 proxy 服务、底层基础服务等）

## Rust 特性简介

### 性能

在 Rust 的设计哲学中，“零成本抽象”是一个非常重要的指导规范，这里可以简单理解为两个要点：

1. **“不用给不需要的东西付费”**，即没有用到的功能，不需要编译和运行时开销
    1.  Rust 没有自动垃圾回收，而是采用资源获取即初始化（RAII）的方式来管理内存，避免了 GC 带来的性能开销。
    2. RAll 核心思想是使用对象来管理内存资源，即在对象的构造函数中分配内存而析构函数中释放内存。该方式确保内存资源在对象在使用时才会分配内存而在不使用时就释放，同时也避免了内存泄漏和悬挂指针等问题。

Rust 更依赖代码生成和静态派发（单态化泛型）等，不鼓励动态派发（Go 的接口值）和运行时反射等。

1. **“需要用到的会做到最优”**，即在最终机器码层面，Rust 会帮助你做到最优（无需考虑性能优化）

Rust 支持和 C/C++ 的零损耗 FFI，代表着 C/C++ 的生态也是 Rust 的生态，即 Rust 都可以使用。

Rust 的编译器后端采用 LLVM，充分利用了C系语言编译器现有的优化能力。

### 安全

Rust 做到内存安全是引入了以下关键的设计：

- Ownership 所有权
- Borrowing 借用
- Lifetime 生命周期

这里的概念实际上和我们日常理解的比较接近，比如：

- 我有一支笔，那么笔就是资源，我现在拥有这支笔的所有权
- 如果你想用这支笔，那么可以向我借用，但这支笔的所有权还是我的（除非转移给你）
- 并且我能决定这支笔能借多久，即多久就是这支笔的生命周期

#### 所有权

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=NGNmNWI2YTU0MDhlNDE1MmE1MTY0ZDIxNTA4NDc4NzFfVzN0aGRldXVtNGRBVXJNWUd6TzVFNFBSYk1SVkdVdUZfVG9rZW46QmUxNGJGWW5Pb3o1UEZ4TkswNmNqUUQ5bjRnXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

这里代码的含义就是申请了一个 String 类型的值为"hello world!"的变量，打印到控制台中。对应到概念中即：

- **变量 x 拥有"hello world!"值的所有权，最后变量 x 在作用域出去后，其值的内存就被释放了**
    - 所以这里也可以看出，Rust 自动管理内存，申请和释放内存的时机在编译时就决定了

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=NDM0N2VjNmQ0ZmFkMzExMmE0NmFlY2I1ZDE1ZTg3OTJfMUl4WHd1OHVLS1M3MXFBMUJxRXVvdW1aZ3V1WUJpbXhfVG9rZW46Rm5tb2J6dU9Zb0VVWkN4YjdvRmNqa3BoblVnXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWQzNzBjZGVjZjcyNTQ4Mzk0ZWFhNDNiMzdlOTUzNzhfQjRKY1E2cWJwZDFMNXlQQmdQbGNnajl2bzVIMjJ3ZzhfVG9rZW46TDM4SmJaVlUxb0dQYm94MHJRTmNGWGVnbldiXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

通过上述代码执行后的错误提示能看出，**s1 拥有值的所有权被转移 s2 后，s1 就无法使用该值了**。

> 其中错误提示也给出该代码的错误原因和后续修复的建议，就涉及到深拷贝和浅拷贝的问题，这里先不关心。

这就是 Rust 里非常重要的概念——**所有权，可理解为管理内存的一组规则**，这里有三条基础规则：

- **Rust 中的每一个值都有一个被称为其所有者（owner）的变量**
    - 比如上面代码中`x`和`s2`就是对应值的 owner
- **值在任意时刻有且只有一个所有者**
    - 避免多次和错误释放等内存安全问题，比如上面代码中若`s1`和`s2`两个都有该值的所有权，则会被多次释放
- **当所有者（变量）离开作用域的时候，这个值将被丢弃（回收）**
    - 保证每次离开作用域自动释放内存，避免内存泄漏问题

将上述规则简单概括就是：每个值有且只有一个拥有所有值的变量，当所有者离开作用域后，该值就失效了。

#### 借用

在上面的例子中可看出，根据所有权规则，若想访问"hello"的值的话，只能通过`s2`访问，显然这样并不利于我们对值复杂的访问需求，虽然这样的规则能够带来内存安全的保障，但若保持这样的访问限制，使用成本就会变得非常高。**所以为了解决这里的问题，Rust 就引入另外一个概念——借用。**

这里我们修改上述例子的代码，来通过借用做到没有所有权的变量也能够访问其值：

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MTc5MjQ5NjdhMzcxNGY0ZWNhMjYyNjcwYmVmMzA4NDhfNW1VME15Z3RrTEVvQWpnS2d1RWZtZWxPZFJUUDZlSG1fVG9rZW46TnoxVGIwTG85b2dQTHd4eU1YdGM4YVpWbkhjXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

**通过借用避免所有权转移，通过共享访问权限（只读且不拥有值），满足多个入口能访问相同所有权值的需求。**

> 注意：看到这里大家可能就会联想到“指针”（通用表示内存地址的类型），但指针和引用在 Rust 中是两个概念。
>
> Rust 中指针和引用都可用来指向内存中的某个值，其主要区别在于两者安全性和生命周期的保证，这里先不关心。

如果我们想要修改这个共享的值，谈论该问题前需要先了解一个新的知识点：

- **「可变」和「不可变」**在 Rust 中是手动显示设置的
- 变量默认是不可变的（比如上面提到的变量均是），若该变量是可变的，则需要用`mut`关键词来声明

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=OGRmMDM2NDFjMWZiYWFmOGJlNjM2OTdhNDY2NmZkMDhfWEJEUUNHeklHRFNrWHdnT3VSNnNMSDEyNkplbWo2cGFfVG9rZW46RlVJSWJ5ZlVwb2JRNEp4RTBGZmNLREd6bkpoXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=YTMxYTIzZmM1MGNhNzMzZDZjZDg5YWNlMjI5MTczOGZfMW1GZW1HSjBSd1NZVFNpenJNWDZqenhSRVVrQVY4NDhfVG9rZW46UzhGY2JxRkEzbzJNNEx4UXY1OGNCd0I0bmpiXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

根据该知识点我们可知前面共享只读访问的例子是，`s2`绑定了`s1`变量的不可变引用。而若想要修改就能推理出：

- 我们需要创建一个可变引用，允许绑定了该可变引用的变量对值进行修改（注意所有权还是在`s1`上）

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MTdmYjg4OWRlMzIzOTRkODUzMzVmN2VmODZmY2RmNDZfbkxhcldjTXZFaU1mV09XandBdDFndFU5ZHZwNEhPM2xfVG9rZW46VDZtcWJkRVZJb1VXa0N4aU9VRWNkdEhhbmpkXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MTlmN2U0NWJkNjk1YjdmNDVhM2M4MDljNzdjOWFiYjRfMllSMnF6REJjMDZGVFFpeDRRTlVHZld3dGlCc1EyZ3BfVG9rZW46UTM3eGI1V0lCb1hWZjR4UVhnYWM2eWV4bk1lXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=NjdkMDQwOGZjNzZhNWVjNmU0ODZiOTgyODkzMTNlMDJfRFJhMHlCZHN5ZDV0b3JXWW14NGt3bndCNVNLYkdzeThfVG9rZW46U1FMRmJCZ1FEb0pDY2p4RDhqZWN0RWJrbkZkXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

上面例子只将`s2`进行了打印，那如果我们尝试把`s1`也跟着一起打印出来，会发生什么呢？

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=Mjk1YmEyMDdjMTY3YzZlY2NmZTRjZTkyYzNhZDNmMzJfeG1YMU5RaGUzZ21VeVNEcFhDZzY1Nm5iR1NpTGlPVzVfVG9rZW46UnJ5SGJ2dWFRbzlaUWZ4WHNBOWNEcTBZbk1nXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

根据错误提示我们大概知道，Rust 好像不允许同时存在可变引用和不可变引用。所以这其实就是借用的规则：

- **同一时刻，要么只能存在一个可变借用，要么任意多个不可变借用**
    - 这样避免了数据竞争的问题
    - 数据竞争可由以下行为造成：
        - 两个或更多的指针同时访问同一数据
        - 至少有一个指针被用来写入数据
        - 没有同步数据访问的机制

而由于存在了引用，让我们来看一个新的问题即“悬垂引用”（指针指向某个值后，该值被释放掉了但指针仍存在）：

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=ODNmMTMxOWVhYjc3MTBjYjQ0OTYxMDc0MzQ4M2ZjZjlfbzg1NDNsY3l1MlpXOFVOV1VmOXp6b1kzUTlDbElIZDZfVG9rZW46SXJia2I0RVhOb2hTZU14NmhjRmM4V2ZPblJoXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDYyNGRhMWVhYjE2MTVjYzA2NmVkYTJhNDgwYTNkMGZfbkdrcjVBVVFON0MwZVZ5dFAzeWcwdkxvRnMxcVo5RUhfVG9rZW46TElwUWJMbklJb2FYQ1l4UVNFbmNCR1ZkbjFnXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

从上述创垂引用的例子可看出，编译报出错误提示该函数返回了一个借用的值，但是已找不到它所借用值的来源。

所以这里也可以看出借用的另一个规则：**引用必须总是有效的。**Rust 会确保引用永远也不会变成悬垂状态。

让我们再回到错误提示中，发现有一个`lifetime parameter` 的关键词，这也就是下面要讲的生命周期。

#### 生命周期

生命周期简而言之就是引用的有效作用域，但相较于前面的概念更加复杂，不过原理其实非常简单：

- **一个资源能借用的时间，不能超过这个资源存在的时间（即能借多久 Owner 说了算）**

这里我们通过一个简单的例子进行说明，左边的代码会得到右边的错误提示：

```Rust
fn main() {
        let r;          // ---------+-- 'a
        {               //          |
            let x = 5;  // -+-- 'b  |
            r = &x;     //  |       |
        } // x被自动释放 // -+       |
        println!("r: {}", r); //   |
}                      // ---------+
```

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=Nzc4YjU5NTU0NmM4MTgzNmQ2ZjBhOTBiOTdmMjJmNDlfUlpvRmVqQ3hBaE9pWWxKa0FGZ1dhZWJXUTUxUlRIemtfVG9rZW46QkpVRmJDeVdnbzFVYUt4OGpMMGNBd3RobmxkXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

实际上错误提示给出的原因非常清晰：

- 在变量 x 自动释放后，变量 r 所绑定的不可变借用的资源已经被自动释放了
- 变量 r 指向了一个被回收的数据的地址，变成了一个悬垂指针，所以就出错了

若按照生命周期规则可知上述修复的思路：**要在变量 r 使用的时候变量 x 还活着，没有被释放**，保证引用安全，即：

```Rust
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

关于生命周期的复杂度实际上会在后续接触了函数、结构体、方法等概念后陡增，这里就先不讨论了（避免劝退）。

只不过还好的是，**在大多数时候，我们无需手动的声明生命周期，因为编译器可以自动进行推导**。而为什么需要了解这个概念，是因为在部分场景下，编译器无法推导出该资源的生命周期时，就需要我们手动标明，用类型来类比下：

- 就像编译器大部分时候可以自动推导类型 <-> 一样，编译器大多数时候也可以自动推导生命周期
- 在多种类型存在时，编译器往往要求我们手动标明类型 <-> 当多个生命周期存在，且编译器无法推导出某个引用的生命周期时，就需要我们手动标明生命周期

#### 无畏并发

这里再额外再介绍一个 Rust 的特性，官方称之为“无畏并发（Fearless Concurrency）”。

众所周知并发编程的痛点之一在于，**容易产生数据竞争的问题**。

> 这里简单解释下数据竞争，其指的是当多个线程同时访问相同的内存位置时，可能会导致未定义的行为。该情况下程序的输出结果可能会因为线程执行的顺序而不同，从而导致程序出现错误。比如对同一变量加一减一的例子。

而在 Rust 中在这里的优势就在于通过前面的所有权和借用规则再加上两个特殊的 Trait（可简单理解为接口 interface），分别是 Send 和 Sync，**你很难写出并发不安全的代码，否则编译会直接报错**，其主要原因如下：

- Rust 无法同时存在多个可变借用或可变和不可变借用共存，即 Rust 里无法同时读写值，在编译期间就能给出相应的错误提示，而**这恰恰是数据竞争的充分必要条件**
- 而同时读写数据对于并发编程是必要的，为了避免编译时的限制，**Rust 提供了运行时可变性的概念**，即原理就是把原本编译时需要检查的可变和不可变之间的互斥性，放在了运行时进行检查（运行时动态确定数据的可变或不可变，且保证在同一时间只有一个线程可以访问一个可变的数据）
- 我们实际使用的都是**可以共存的不可变引用，写值的时候再做互斥性检查（或使用原子操作）**。这样就可在不违背 Rust 借用检查的前提下，提供并发读写的能力，同时也强制了用户必须使用这些同步原语来读写数据
- 通过 **Send 和 Sync 两个标记来对数据有精细的并行访问控制**，比如本地线程相关的数据，不希望其他线程访问，则可以标记它非 Send 非 Sync

1. Rust 通过上面提到的借用检查、运行时可变性以及 Send/Sync 标记让安全并发非常容易，并为开发者带来了极大的便利。

而且前面也提到 Rust 的原则之一就是零成本抽象，所有的这些 Ownership、Borrowing、Send/Sync 标记的检查全部均在编译时（静态检查时）完成的，**而运行时根本没有任何的性能损耗**，相比其他一些编程语言（java、python等），可能会在运行时执行更多的动态检查或使用锁等机制来保证并发操作的正确性，这些额外的运行时开销可能会导致性能下降。

### 协作

这里为什么说 Rust 很适合协作？非常重要的一点在于：

- **Rust 编译器决定了较高的质量下限，甚至可以说你可以完全信任别人的代码！**

在项目协作的 code review 中，不用过于担心潜在的各种坑，可实现更加高效的开发、review、merge 流程。

并且 Rust 作为一门工程实践出来的语言，还提供了智能的编译器、完善的文档、齐全的工具链、成熟的包管理等，因此我们**只需要专注逻辑功能的实现**，而编译器的检查帮我们完成了内存安全、并发安全等问题。

### 与 Go 的相似点

> 两种语言的共同目标是什么呢？
>
> - Rust 是一种专注于安全性和性能的低级静态类型多范式编程语言。—Gints Dreimanis
> - Go 是一种开放源代码编程语言，可轻松构建简单，可靠和高效的软件。—Golang.org

Rust 和 Go 均作为现代强大且被广泛采用的编程语言，还是有许多相似的地方：

- **内存安全方面**：都以不同的方式处理内存安全问题，但是两者的目的都是要比其他有关内存管理的语言更聪明，更安全，都希望使用消息通讯而非共享内存，并帮助你编写正确且性能良好的程序
- **快速，紧凑的可执行文件**：都是编译语言，意味着程序直接转换为可执行的机器代码，因此你可以将程序作为单个二进制文件进行部署，且与解释型语言不同，无需随程序一起分发解释器，大量库和依赖项
- **通用语言**：都是功能强大，可扩展的通用编程语言，可使用它们来开发各种现代软件，从 Web 应用程序到分布式微服务，或者从嵌入式微控制器到移动应用程序。两者都具有出色的标准库和蓬勃发展的第三方生态系统，以及强大的商业支持和庞大的用户群
- **务实的编程风格**：都不是纯函数式语言，也不是全面面向对象的语言。相反，尽管 Go 和 Rust 都具有与函数和面向对象的编程相关的功能，但它们都是务实的语言，旨在以最合适的方式解决问题（如无构造函数，使用组合抛弃继承，函数是一等公民，不允许函数重载、错误处理基于值而非异常等），而不是强迫采用特定的处理方式

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=Nzc2ZGQzM2ZlOGFmMGZlMDcyMmViMmMxMjg1N2VmNmRfNnNLcVdhcGFJNTRUOGJyRjRuTkhMZmhoSnd1MHpnTVpfVG9rZW46UGd1ZGJENXZ6b3hrNml4djBUdGNOQUo1bkJkXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=N2FmNzdhZDY0YTc5YjcyMzBhNWNiMTQyOGU4NWI4YTFfM0xzUkttTDlwVGtKOVI4UzhteXNCRWZtZEZjS1FZeUdfVG9rZW46VU4xSmJERjVDb254RkR4V25tMGNFNzRsbmloXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

## Rust 语法简介&差异

> **语法小抄：** **[Rust Cheat sheet](https://docs.google.com/document/d/1kQidzAlbqapu-WZTuw4Djik0uTqMZYyiMXTM9F21Dz4/edit#heading=h.gjdgxs)**

### 变量绑定与解构

- 变量命名：需要遵循 [Rust 命名规范](https://course.rs/practice/naming.html)
    - 对于 **type-level** 的构造倾向于使用**驼峰命名法**
    - 对于 **value-level** 的构造使用**蛇形命名法**
- 变量绑定：也可称为赋值，但是绑定能更加清晰地贴合 Rust **所有权**的核心原则
- **🌟变量可变：Rust 变量默认是不可变的，若可变（即允许值变化）则需要使用** **`mut`** **关键词指定**
- 未使用的变量：若希望告诉 Rust 不要警告未使用的变量，则需要使用下划线作为变量名的开头
- **🌟变量解构：** **`let`** **表达式不仅仅用于变量的绑定，还能进行复杂变量的解构**
- 变量和常量之间的差异：常量默认不可变，而且自始至终不可变，不允许使用 `mut`，使用`const`关键词声明
- **🌟变量遮蔽：允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的**

```Rust
fn main() {
    // 1. 变量可变
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);

    // 2. 变量解构
    let (a, mut b): (bool, bool) = (true, false);
    println!("a = {:?}, b = {:?}", a, b);
    b = true;
    println!("a = {:?}, b = {:?}", a, b);

    // 3. 变量遮蔽
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let x = x + 1;
    {
        // 在当前的花括号作用域内，对之前的x进行遮蔽
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }
    println!("The value of x is: {}", x);
}
```

```go
func main() {
    // 1. 变量可变
    var x = 5
    println("The value of x is: %d", x)
    x = 6
    println("The value of x is: %d", x)

    // 2. 变量解构
    var a, b bool = true, false
    println("a = %v, b = %v", a, b)
    b = true
    println("a = %v, b = %v", a, b)

    // 3. 变量遮蔽
    // 这里也会报错，提示该名称变量已被重复声明
    var x = 5
    x = x + 1
    {
       // 这里会输出？
       x := x * 2
       println("The value of x in the inner scope is: %d", x)
    }
    println("The value of x is: %d", x)
}
```

### 语句与表达式

- Rust 的函数体是由一系列语句组成，最后由一个表达式来返回值
- 对于 Rust 语言来说，**需要明确语句（statement）和表达式（expression）的概念**
    - 因为**基于表达式是函数式语言**的重要特征，表达式总要返回值
    - 而在 golang 中实际没有特别注意语句和表达式的概念，不同在于 Go 的赋值操作是一个表达式而不是语句

```Rust
fn add_with_extra(x: i32, y: i32) -> i32 {
    let x = x + 1; // 语句
    let y = y + 5; // 语句
    x + y // 表达式
}

fn main() {
    let y = {   // 语句块表达式
        let x = 3;
        x + 1       // 表达式不能包含分号
    };
    println!("The value of y is: {}", y);
}

fn ret_unit_type() {
    let x = 1;
    // if 语句块也是一个表达式，因此可以用于赋值，也可以直接返回
    // 类似三元运算符，在Rust里我们可以这样写
    let y = if x % 2 == 1 { // 若y没有使用Rust会给出提示，但不会报错
        "odd"
    } else {
        "even"
    };
}
```

```go
func addWithExtra(x int, y int) int {
    x = x + 1
    y = y + 5
    return x + y
}

func main() {
    y := func() int {
       x := 3
       return x + 1
    }()
    println("The value of y is: %d\n", y)
}

func retUnitType() {
    x := 1
    var y string   // 若y没有使用，这里会直接报错
    if x%2 == 1 {
       y = "odd"
    } else {
       y = "even"
    }
}
```

### 所有权

在前面已经介绍了所有权的规则和基本概念，这里我们重点介绍下之前没有介绍所有权细节之浅拷贝和深拷贝。

#### 前置知识

讲述之前我们这里需要先简单了解些前置知识

1. **栈与堆**

栈和堆的核心目标就是为**程序在运行时提供可供使用的内存空间**：

- 对于大小已知且固定内存空间大小的数据，需要将它存储在栈上
- 对于大小未知或者可能变化的数据，需要将它存储在堆上
    - 其分配的具体过程为操作系统在堆的某处找到足够大的空间，把它标记为已使用，并返回一个表示该位置地址的指针, 该过程被称为在堆上分配内存，而指针因为大小已知且固定，所以该指针会被推入栈中
    - 在后续使用过程中，**将通过栈中的指针，来获取数据在堆上的实际内存位置，进而访问该数据**

因为堆上的数据缺乏组织，因此堆数据若不能保证被释放则会造成内存泄漏问题（即数据将永远无法被回收），所以 Rust 所有权规则通过跟踪数据的分配和释放，保证该部分内存安全。

1. **Rust 中的 String 类型**

- `str`是 Rust 中的基本类型（字符类型），而`String`是复合类型
- `let s ="hello"`：`s` 是被硬编码进程序里的字符串值（类型为 `&str` ），被称为字符串字面值
    - 虽然字符串字面值较方便，但是并不适用于所有场景：
        - **字符串字面值是不可变的**，因为被硬编码到程序代码中（栈中）
        - 并非所有字符串的值都能在编写代码时得知，如需要在控制台动态输入字符场景
- 为解决用字符串字面值导致字符串不可变和无法动态的问题，为此提供 String 类型，该类型会被分配到堆上
    - `let s = String::from("hello")`创建 String 类型，其中`::`是一种调用符，`from`是 String 的方法

> String 类型的组成与 Go 中的 string 类似，由**存储在栈中的堆指针**、**字符串长度**、**字符串容量**共同组成，其中**堆指针**是最重要的，它指向了真实存储字符串内容的堆内存，至于长度和容量，容量是堆内存分配空间的大小，长度是目前已经使用的大小。

#### 拷贝（浅拷贝）与克隆（深拷贝）

回到前面介绍的例子：

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MDZmMjVhZmE1YjVmN2RiMmE3NWI5MzA1MmIyZTQ2ZmFfc2dtMVVrUG5JRGFHcjRpa3o1MndHb05UMnR0MlZTWlNfVG9rZW46UXY1WWJhRUpWb1AySTV4bGhGQmNlRjk1blZjXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDdjOTRiYjJmMTlhMjE5YWI3NzNhYjlkNDZmOTE1OTZfZVl2c1U4QjJkRWFHUHJnVFpzdE9nbk9PTEE5Z1V4MUFfVG9rZW46TzZxbGJsMjBNb2FISkp4QmVlOWNnOVJMbkdnXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

在错误提示中我们可以发现有两个关键词`Copy`和`Clone`，实际上这两个关键词就分别对应着：

- Copy -> **拷贝（浅拷贝，shallow copy）**
- Clone ->**克隆（深拷贝，deep copy）**

在介绍具体拷贝和克隆之前，我们先来理解下上面的例子，这里需要记住一个新的知识点，即：

- **对于基本类型（存储在栈上），Rust 会自动拷贝，而对于复合类型（存储在堆上），Rust 则不会自动拷贝**

所以这就是为什么下面代码能够正常编译，是因为基本类型的值进行自动拷贝后，拥有了新的栈内存（所有权）：

> 注意这里实际上没有深浅拷贝的区别，可以理解成在栈上做了深拷贝。

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=OWU2Yzg4MjJjMDRmNWYzMTFlZjBkMGUwN2RiYTQ5NWJfeHZCN1NVNnM3a0hsUVROMDV5TVZCb0RuSzBhbUVkQzhfVG9rZW46V3J1VWJJRFlhb2dXMTN4ZHRXQ2N5dHUybjVmXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

根据上述知识点，我们来进行理解：

1. **复合类型为什么不能自动拷贝**

String 数据存储在堆上，若进行自动拷贝（浅拷贝）即只会拷贝指向堆上数据的指针，而不会拷贝实际的数据。这可能导致多个变量指向同一块内存（实际上就不满足堆上值的所有权规则之**一个值只允许有一个所有者**），若某个变量将指向的内存数据进行释放，此时其他变量的引用就是无效引用且可能会造成多次释放的内存安全问题。所以在 Rust 中复合类型就不会发生自动拷贝。

1. **String 的所有权转移**

- 当变量 s1 赋予变量 s2 后，Rust 就认为 s1 不再有效，因此无需在 s1 离开作用域后 drop 任何东西
    - 即**把所有权从变量 s1 转移给了变量  s2**
- 当拷贝 String 类型的指针、长度和容量而不拷贝数据，虽然听起来像浅拷贝，但因为 Rust 同时使第一个变量 s1 无效了，所以该操作**被称为移动（move），而不是浅拷贝**
    - ![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MGEyMDNkMjlhNzM0ODkyOWUwY2I2Y2EyYjdiNGM3ZmFfZHY5NHRNbzI4bklrUTlDdDRpUVpBTGphU0dDUTFPN2RfVG9rZW46Rzhmb2J5c29sbzJFZEp4UnhqV2NjbTdFblN1XzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

而如果将上述例子替换成前面的字符串字面值（`&str`），以下代码能够正常编译：

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=YThiNmViM2VmYzRjNzhkZTYxOTc4OTk2ZGU5ZGM4ZjNfNm1HVkdsRlQyQ3kyWUVoQWZEOEFsV0RTak4wWmlaTndfVG9rZW46Uzdvc2JNamZnb1A4TGR4NUpYbGM4QzBSbmFmXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

##### 克隆（深拷贝）

- **Rust 永远也不会自动创建数据的 “深拷贝”**
    - 因此，任何**自动**的复制都不是深拷贝，可以被认为对运行时性能影响较小
- 若我们确实需要深度复制`String`中**堆上数据**，而不仅是栈上的数据，可使用一个叫做`clone`的方法
    - 需要注意小心使用 `clone` ，因为会极大的**降低程序性能**
- String 类型可理解实现了`Clone`的特征（特征可先简单理解为接口 interface），拥有了克隆的能力

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MzIzNjYyODUxOTcyZGVhNjgxOGQ4ZDViYjBhYjg1YmJfQnExU3dlcXVVOEtWZnVDTnRxM0F1ZG1qUFBwa2wwOHhfVG9rZW46QjNhcWJVemJPb1dXWkl4V0pnamNpMGl4bkNkXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

##### 拷贝（浅拷贝）

- 浅拷贝只发生在栈上，因此性能很高
- 类似`Clone`特征，若想拥有拷贝的能力，则需要实现 `Copy` 特征，而什么类型可 Copy 是有限制的
    - 基本类型默认实现了该特征，包含任何基本类型的组合
    - 不需要分配内存或某种形式资源的类型是可 Copy，比如前面替换`&str`的例子用到了不可变引用 `&T`

> 注意: 可变引用 `&mut T` 是不可以 Copy 的。

#### 函数传值与返回

- 就跟 `let` 语句一样，将值传递给函数或者函数返回值，也一样会发生**「移动」或者「复制」**

    - 例子1：将值传递给函数

        ```Rust
      fn main() {
          let s = String::from("hello");  // s 进入作用域
      
          takes_ownership(s);             // s 的值移动到函数里 ...
          // ... 所以到这里不再有效
          // 所以这里若使用变量 s 则会报错
          println!("{}",s);
      
          let x = 5;                      // x 进入作用域
      
          makes_copy(x);                  // x 应该移动函数里，
          // 但 i32 是 Copy 的，所以在后面可继续使用 x
      
      } // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
        // 所以不会有特殊操作
      
      fn takes_ownership(some_string: String) { // some_string 进入作用域
          println!("{}", some_string);
      } // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放
      
      fn makes_copy(some_integer: i32) { // some_integer 进入作用域
          println!("{}", some_integer);
      } // 这里，some_integer 移出作用域。不会有特殊操作
      ```

    - 例子2：函数返回值

      ```Rust
      fn main() {
          let s1 = gives_ownership();         // gives_ownership 将返回值
          // 移给 s1
      
          let s2 = String::from("hello");     // s2 进入作用域
      
          let s3 = takes_and_gives_back(s2);  // s2 被移动到
          // takes_and_gives_back 中,
          // 它也将返回值移给 s3
      
          // 所以若这里使用 s2 则会报错，因为所有权已经转移到 s3
          println!("{}",s2);
      
      
      } // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
        // 所以什么也不会发生。s1 移出作用域并被丢弃
      
      fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
          // 调用它的函数
      
          let some_string = String::from("hello"); // some_string 进入作用域.
      
          some_string                              // 返回 some_string 并移出给调用的函数
      }
      
      // takes_and_gives_back 将传入字符串并返回该值
      fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域
      
          a_string  // 返回 a_string 并移出给调用的函数
      }
      ```

### 引用与借用

- 借用就是为了避免所有权的转移。**获取变量的引用，称之为借用（borrowing）**

- 常规引用是一个指针类型，指向对象存储的内存地址，`&` 符号即引用，允许使用值但不获取所有权（不会丢弃）

    - 引用指向的值默认也是不可变的

        ```Rust
      fn main() {
          let s1 = String::from("hello");
          let len = calculate_length(&s1);
          println!("The length of '{}' is {}.", s1, len);
      }
      
      fn calculate_length(s: &String) -> usize { // s 是对 String 的引用
          s.len()
      } // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权，
      // 所以什么也不会发生
      ```

      ```Go
      func main() {
          s1 := "hello"
          s1Len := calculateLength(s1)
          println("The length of '%s' is %d.", s1, s1Len)
      }
      
      func calculateLength(s string) int {
          return len(s)
      }
      ```

    ![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MDg2NTFmMzNiZWNkZGYzOTQ3NzViOGIwMmU2ZjExYThfdGdrdEh4Sm1DQ0dlMU9VakZNd3RQdFNWRGlBbUFIemRfVG9rZW46RzNsTmIwbFVib2tvcWt4NW8wNGNiSDhrbmJjXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

    - 在使用具体值时需要解引用

       ```Rust
      fn main() {
          let x = 5;
          // y 获取了变量x的引用（x将资源借用给了y）
          let y = &x;
          // assert_eq 可理解为 Rust 中的断言函数，比较左边和右边是否相等
          assert_eq!(5, x);
          // 若需要取出 y 指向的数据，则必须使用 *y 来解引用
          // 若直接用 y 进行比较，则会报错提示
          assert_eq!(5, *y);
      }
      ```

      ```Go
      func main() {
          x := 5
          y := &x
          println(x)
          println(*y)
      }
      ```


- 若需要对引用进行修改，则需要通过`&mut`指定为可变引用

    ```Rust
    fn main() {
        let mut s = String::from("hello");
    
        change(&mut s);
        println!("{}", s);
    }
    
    fn change(some_string: &mut String) {
        some_string.push_str(", world");
    }
    ```

    ```Go
    func main() {
        s1 := "hello"
        change(&s1)
        println(s1)
    }
    
    func change(s *string) {
        *s += ", world"
    }
    ```


- 🌟借用规则

    - 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用

    ```Rust
      fn main() {
          let mut s = String::from("hello");
      
          let r1 = &mut s;
          let r2 = &mut s;
      
          println!("{}, {}", r1, r2);
      }
    ```

    ![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=MjYzYzUyMzliYmNkMjU5NGEyZjJiNGRhZDQyM2IyY2FfaE5lSHpaRlplaGZuSGREZzZSZ3ZNOHU0Rm9wR29OY0lfVG9rZW46RHhJaGJ5a1Jlb3FlY2p4ZVNXMGNJZzl5bk1oXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

    ```Rust
      fn main() {
          let mut s = String::from("hello");
      
          let r1 = &s; // 没问题
          let r2 = &s; // 没问题
          let r3 = &mut s; // 大问题
      
          println!("{}, {}, and {}", r1, r2, r3);
      }
    ```

    ![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=OTIwZTRkOWZhYzZiNmYwOWU1ZDk0NWViYmRhNGY2NzRfc2xhVzhjMnRSekFQY1cwaE5xRlVacDg0U0l4QmROcEhfVG9rZW46V2dlTWJRYk9sbzliV1N4cnZpd2NHTTJFbmtiXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

  - 引用必须总是有效的

### 复合类型

#### 数组与切片

- Rust 的数组是基本类型，其长度固定且依次线性排列相同类型的元素，与 Go 类似

    ```Rust
    fn main() {
        // 编译器自动推导出one的类型
        let one = [1, 2, 3];
        // 显式类型标注
        let two: [u8; 3] = [1, 2, 3];
        let blank1 = [0; 3];    // [初始值;数组长度]，这里就为固定长度为3初始值为0的数组
        let blank2: [u8; 3] = [0; 3];  
    
        // arrays是一个二维数组，其中每一个元素都是一个数组，元素类型是[u8; 3]
        let arrays: [[u8; 3]; 4] = [one, two, blank1, blank2];
    }
    ```

    ```Go
    func main() {
        // 自动推导出 one 的类型
        one := [3]int{1, 2, 3}
        // 显式类型标注
        var two [3]int = [3]int{1, 2, 3}
        blank1 := [3]int{0}
        var blank2 [3]int = [3]int{0}
    
        _ = [4][3]int{one, two, blank1, blank2}
    }
    ```

- Rust 中的切片是对集合一定范围的访问，即允许你引用集合中部分连续的元素序列，而不是引用整个集合

    - **与 Go 的切片需区分，前者是指动态数组，而后者指一个长度和指向某个连续的内存区域的指针，无法扩展**
    - 而在 Rust 的动态数组是用 Vector 来表示，即 Go 的切片同时包含了 Rust 的 Vec 和 "slice"

- Rust 创建切片是使用`[开始索引..终止索引]`来声明

```Rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];  // 等于 &s[..5]
    let world = &s[6..11]; // 等于 &s[6..]，11是字符串长度
    let all_str = &s[..];     // 等于 &s[0..len]
    println!("{},{},{}", hello, world, all_str)
}
```

```go
func main() {
    s1 := "hello world"
    hello := s1[0:5]
    world := s1[6:11]
    allStr := s1[:]
    println(hello, world, allStr)
}
```

#### 字符串

- 在 Go 中，`[]byte`与`string`的实现存在差异，所以导致了两者之间相互转换需要进行深拷贝
- 在 Rust 中，存在两种字符串类型`str`和`String`，其实现分别和`[u8]`以及`Vec<u8>`完全相同，**所以在相互转换的时候不需要进行拷贝**
    - 但 Rust 中要求字符串里的数据必须是 UTF8，所以在字符切片转换成字符串时，需要进行UTF8检查
- `str`的含义是内存里的一块长度未知的字符串，而`&str`是**固定长度字符串的引用**，由两部分组成：指向字符串序列的指针和字符串长度值。`String` 是**堆分配且可动态增长的字符串**，被存储为由字节组成的 vector
    - 需要注意，由 &str 转为 String 的性能开销高于 String 转为 &str，因为前者需要在堆上分配内存去复制 str，而后者只需要栈上复制 String 指针即可

```Rust
const data: &str = "data";

fn main() {
    let byte_data: &[u8] = data.as_bytes(); // 不需要检查，直接转换
    let s = std::str::from_utf8(byte_data).unwrap(); // 需要检查
    let s = unsafe {
        std::str::from_utf8_unchecked(byte_data) // 使用 unsafe 方法绕过检查
    };
    let string: String = s.to_string(); // 需要深拷贝
    let s: &str = &string;
}
```

#### 元组

- 元组由多种类型组合到一起形成的，元组的长度和元素的顺序都是固定的
- 使用括号 `()` 来创建元组，而每个元组自身又是一个类型标记为 `(T1, T2, ...)` 的值，其中 T 是元素类型
- Rust 中函数不支持返回多个值，但可使用元组来实现返回多个值的效果

```Rust
fn main() {
    let s1 = String::from("hello");

    // 解构方式使用
    let (s2, len) = calculate_length(s1);
    println!("The length of '{}' is {}.", s2, len);

    // 非解构方式使用
    let str_info = calculate_length(s2);
    println!("The length of '{}' is {}.", str_info.0, str_info.1);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度
    (s, length)
}
```

#### 结构体

- 结构体的概念与其他编程语言一致，其语法也类似

    ```Rust
    struct User {
        active: bool,
        email: String,
    }
    
    fn main() {
        let user1 = User {
            email: String::from("someone@example.com"),
            active: true,
        };
    
        // 注意若字段需要变化，也需要指定该结构体为可变的
        let mut mut_user = User {
            email: String::from("someone@example.com"),
            active: true,
        };
        mut_user.email = String::from("anotheremail@example.com");
    
    
        let user2 = User {
            active: user1.active,
            email: String::from("another@example.com"),
        };
        // 等价于下面代码
        // let user2 = User {
        //     email: String::from("another@example.com"),
        //     ..user1
        // };
    }
    ```

- 特殊结构体

    - 结构体的字段可以没有名称，被称为“元组结构体”

        ```Rust
          struct Color(i32, i32, i32);
          struct Point(i32, i32, i32);
      
          let black = Color(0, 0, 0);
          let origin = Point(0, 0, 0);
        ```

    - 没有任何字段和属性（通常只关心其行为），被称为“单元结构体”

        ```Rust
      struct AlwaysEqual;
      
      let subject = AlwaysEqual;
      
      impl SomeTrait for AlwaysEqual {
      }
        ```

- 🌟**结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段**

    ```Rust
    #[derive(Debug)]
    struct User {
        active: bool,
        username: String,
        email: String,
        sign_in_count: i64,
    }
    
    fn main() {
        let user1 = User {
            email: String::from("someone@example.com"),
            username: String::from("someusername123"),
            active: true,
            sign_in_count: 1,
        };
        let user2 = User {
            active: user1.active,
            username: user1.username,
            email: String::from("another@example.com"),
            sign_in_count: user1.sign_in_count,
        };
        // 这里 active 由于是基本类型实际发生了自动拷贝
        println!("{}", user1.active);
        // 下面这行会报错，因为 user1 的所有权已经转移了
        println!("{:?}", user1);
    }
    ```

- 生命周期能确保结构体的作用范围要比它所借用的数据的作用范围要小，所以使用引用时就必须加上生命周期

    ```Rust
    struct User {
        username: &str,
        email: &str,
        sign_in_count: u64,
        active: bool,
    }
    
    fn main() {
        let user1 = User {
            email: "someone@example.com",
            username: "someusername123",
            active: true,
            sign_in_count: 1,
        };
    }
    ```

    ![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=ODkwYjYyMWFmNjNkYzRkYzc3YmM5NzZlZWQ5ODIyZWNfYng2V0Y1REZMdzM2Uk1ROHRsWjZ1aTE0bXlVc245cUxfVG9rZW46SW9iaGI5aUdNb1hGNHR4WE55dGNvWVNLbmZoXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

    - 修复如下：

     ```Rust
    struct User<'a> {
        username: &'a str,
        email: &'a str,
        sign_in_count: u64,
        active: bool,
    }
     ```

#### 枚举

- **Rust 拥有单独的枚举类型**（Go 没有），会包含所有可能的枚举成员, 而枚举值是该类型中的具体某个成员的实例

- **任何类型的数据都可以放入枚举成员中**，如字符串、数值、结构体甚至另一个枚举

    ```Go
    // golang 中没有单独的枚举类型，通常通过该方式来模拟
    type Color int
    
    const (
        Blue Color = iota
        Red
    )
     ```

    ```Rust
    enum Character {
        A,
        // 直接将成员作为枚举值，没有关联任何数据
        B,
        C(i32),
        // 持有数据类型的成员
        D(i32, i32),
        // 持有元组类型
        E(String),
        // 持有String类型
        F { x: i32, y: i32 },  // 持续结构体类型
    }
    
    // 拥有隐式辨别值（implicit discriminator，类似 itoa 即0开始）的枚举
    enum Number {
        Zero,
        One,
        Two,
    }
    
    // 拥有显式辨别值（explicit discriminator）的 enum
    #[derive(Debug)]
    enum Color {
        Red = 0xff0000,
        Green = 0x00ff00,
        Blue = 0x0000ff,
    }
    
    
    fn main() {
        let a = Character::A;
        let b = Character::C(1);
        let c = Character::D(1, 2);
        let e = Character::E("hello".to_string());
        let f = Character::F { x: 1, y: 2 };
        let zero = Number::Zero as u8; // 0
        let red = Color::Red as i32;  // 16711680
    }
     ```

- Rust 通过**引入** **Option 枚举用于处理空值** **（即取代了 Go 中的 nil）**，类似的还有 Result 枚举用于错误处理

    - Rust 使用 Option 取代空值，所有可能产生空值的返回都会被包装成 Option<T> 类型，且**在使用的时候需通过模式匹配或特殊方法去显式解构去获取到值**，该设计也在一定程度上避免操作空值导致的 panic 等常见错误

    - 其中 Option 和 Result 枚举类型都在`prelude` 中（Rust 标准库，会将最常用的类型、函数等自动引入）

        ```Rust
      fn plus_one(x: Option<i32>) -> Option<i32> {
          match x {           // 模式匹配语法
              None => None,               // 表示空值的处理
              Some(i) => Some(i + 1),// 表示有值的处理
          }
      }
      
      fn positive_num(x: i32) -> Result<i32, String> {
          if x > 0 {
              return Ok(x);
          }
          Err("negative_num".to_string())
      }
      
      fn main() {
          let five = Some(5);
          let six = plus_one(five)?; // 此处'?'这里可理解为解构处理，后续在错误处理会详细解释
          let none = plus_one(None);
          let positive_num = positive_num(10)?;
      }
       ```

       ```go
      func plusOne(x *int) *int {
          if x == nil {
             return nil
          }
          *x++
          return x
      }
      
      func positiveNum(x int) (int, error) {
          if x > 0 {
             return 0, nil
          }
          return 0, errors.New("positive number")
      }
      
      func plusNum(x, y int) (int, error) {
          x, err := positiveNum(x)
          if err != nil {
             return 0, err
          }
          y, err = positiveNum(y)
          if err != nil {
             return 0, err
          }
          return x + y, nil
      }
      ```


### 流程控制

- Rust 中流程控制的关键词和基本用法与 Go 类似，如`if`、`for`、`while`、`contine`、`break`、`loop`

- 其中 Rust 存在一些特殊用法和 Go 有差异的地方

    - **if 语句块是表达式，**能够通过 if 表达式的返回值进行赋值

        ```Rust
      fn main() {
          let condition = true;
          let number = if condition {
              5
          } else {
              6
          };
          println!("The value of number is: {}", number);
      }
        ```

    - for 循环语法为`for 元素 in 集合`，**同时需要考虑所有权**

        ```Rust
      fn main() {
          for i in 0..5 {    // 等价于 for i:=0;i<5;i++{}
              println!("{i}")
          }
          for i in 0..=5 {   // 等价于 for i:=0;i<=5;i++{}
              println!("{i}")
          }
          let collection = [0; 3];
          // 转移所有权
          for item in collection { // 等价于 for item in IntoIterator::into_iter(collection) {}
              println!("{item}")
          }
          // 不可变借用
          for item in &collection { // 等价于 for item in collection.iter() {}
              println!("{item}")
          }
          let mut mut_collection = [0; 3];
          // 可变借用
          for item in &mut mut_collection { // 等价于 for item in collection.iter_mut() {}
              println!("{item}")
          }
      }
        ```

    - break 可以单独使用，**也可以带一个返回值**，有些类似 `return`，loop 也是一个表达式且可返回值

         ```Rust
      fn main() {
          let mut counter = 0;
          let result = loop {
              counter += 1;
              if counter == 10 {
                  break counter * 2;
              }
          };
          println!("The result is {}", result);
      }
        ```

### 方法

- Rust 使用`impl`来定义方法且需要指定的结构体，可用`self`来指代结构体实例，**注意其拥有所有权的概念**
    - `self` 表示结构体实例所有权转移到该方法中（该用法较少）
    - `&self` 等价于`self:&Self`（`Self`指代结构体类型）表示该方法对结构体实例的不可变借用
    - `&mut self` 表示结构体实例的可变借用

```Rust
struct Point {
    x: f64,
    y: f64,
}

impl Point {
    // 在 impl 中且没有 self 的函数被称之为关联函数
    // Rust 中有一个约定俗成的规则，使用 new 来作为构造器的名称
    // 因为是函数，所以不能用 . 的方式来调用，需要用 :: 来调用
    pub fn new(x: f64, y: f64) -> Point {   // 这里返回的 Point 可用 Self 替代
        Point { x, y }
    }

    // 通常方法跟字段同名，往往适用于实现 getter 访问器
    // 方法调用的方式是用 .
    // 注意这里的'&self'是不可变借用
    pub fn x(&self) -> f64 {
        return self.x;
    }

    // 注意这里的‘&mut self‘是可变借用
    pub fn set(&mut self, x: f64, y: f64) {
        self.x = x;
        self.y = y;
    }
}


impl Point { // Rust 允许同时多个 impl 定义
    pub fn print(&self) -> () { // ()被称之为单元类型，用来返回空类型
        println!("{},{}", self.x, self.y)
    }
}
```

```go
type Point struct {
    x, y float64
}

func New(x, y float64) Point {
    return Point{x, y}
}

// golang 中方法不能和字段重名
func (p Point) GetX() float64 {
    return p.x
}

func (p *Point) Set(x, y float64) {
    p.x, p.y = x, y
}
```

### 模式匹配

- 类似 Go 中的 switch 匹配，Rust 中的模式匹配更强大，**支持各种模式类型的匹配，并常用来对包装类型进行解构**

- Rust 最常用的模式匹配语法就是`match`和`if let`，使用时需要注意

    ```Rust
    match target {
        模式1 => 表达式1,
        模式2 => {
            语句1;
            表达式2
        },
        _ => 表达式3
    }
    ```

    ```Rust
    if let 模式1 = target {
        表达式
    }
    ```

    ```Rust
    fn main() {
        let v = Some(3u8);
        match v {
            Some(3) => println!("three"),
            _ => (),
        }
        // 等价于
        if let Some(3) = v {
            println!("three");
        }
    }
     ```

    ```Go
    type Option int
    const (
        Some Option = iota
        None
    )
    
    func main() {
        v := Some
        switch v {
        case Some:
           fmt.Println(v)
        default:
        }
        if v == Some {
           fmt.Println(v)
        }
    }
    ```

    - `match`匹配必须穷举所有可能，其每一个分支都必须是一个表达式且分支表达式返回值类型必须相同

    - 可使用`X|Y`来进行或逻辑运算，也可使用通配符`_`来承载匹配的默认情况，类似 Go 中的`default`

- Rust 也提供了一个非常实用的宏`matches!`，可将一个表达式跟模式进行匹配，然后返回匹配 bool 结果

    ```Rust
    let foo = 'f';
    assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));
    
    let bar = Some(4);
    assert!(matches!(bar, Some(x) if x > 2));
    ```


| Rust 类型匹配                                                | Rust 类型解构                                                |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `fn main() {    *// 匹配字面值*    let x = 1;    match x {        1 => println!("one"),        *// 这里通过'..='匹配值的范围*        2..=10 => println!("two-ten"),        _ => println!("anything"),    }    *// 匹配命名变量*    let x = *Some*(5);    let y = 10;    match x {        *// 这里通过'|'语法做到单分支匹配多个模式*        *Some*(50) | *Some*(10) => println!("Got 50"), *// 注意这里是无法匹配 Some(5) 的*        *Some*(y) => println!("Matched, y = {:?}", y),        _ => println!("Default case, x = {:?}", x),    }    *// 匹配守卫提供的额外条件*    let num = *Some*(4);    match num {        *Some*(x) if x < 5 => println!("less than five: {}", x),        *Some*(x) => println!("{}", x),        *None* => (),    } }     ` | `struct Point {    x: i32,    y: i32, } enum Message {    *Quit*,    *Move* { x: i32, y: i32 },    *Write*(String),    *ChangeColor*(i32, i32, i32), } fn main() {    *// 解构结构体*    let p = Point { x: 0, y: 7 };    match p {        Point { x, y: 0 } => println!("On the x axis at {}", x),        Point { x: 0, y } => println!("On the y axis at {}", y),        Point { x, y } => println!("On neither axis: ({}, {})", x, y),    }    *// 解构枚举*    let msg = Message::*ChangeColor*(0, 160, 255);    match msg {        Message::*Quit* => println!("The Quit variant has no data to destructure."),        Message::*Move* { x, y } => println!("Move in the x direction {x} and in the y direction {y}"),        Message::*Write*(text) => println!("Text message: {text}"),        Message::*ChangeColor*(r, g, b) => println!("Change the color to red {r}, green {g}, and blue {b}"),    }    *// 解构结构体和元组*    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });    *// 解构数组——定长数组*    let arr: [u16; 2] = [114, 514];    let [x, y] = arr;    *// 解构数组-不定长数组*    let arr: &[u16] = &[114, 514];    if let [x, ..] = arr {        assert_eq!(x, &114);    }    *// 匹配宏*    let arr: &[u16] = &[];    assert!(matches!(arr, [..])); }` |



### 泛型

- Rust 的泛型用法与 Go 类似，都需要在**使用前对其进行声明**，使用`<T>`标识为泛型
    - 其中 T 为泛型参数，名称可任起，出于惯例通常为 T，且越短越好
- Rust 在 1.51 版本引入了 const 泛型，实现了对针对值的泛型，通常处理数组长度的问题
- Rust 中的泛型是**零成本的抽象，即运行时无开销**，但牺牲了编译速度和最终生成文件大小
    - 具体原因是 Rust 通过在编译时进行**泛型代码的单态化来保证效率**（单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程）

| *结构体使用泛型* | `struct Point<T, U> { *// x,y 必须保证类型相同*    x: T,      y: T,    z: U,   }` | `fn main() {    let p1 = Point { x: 5, y: 10, z: 5.0 };    println!("p.x = {}", p1.x());    let p2 = Point { x: "Hello", y: "World", z: 'c' };    let p3 = p1.mixup(p2);    println!("p3.x = {}, p3.z = {}", p3.x, p3.z);     let s1 = *Some*(5);    let s2 = *Some*("a");    match s1 {        *Some*(i) => println!("{i}"),        *None* => *None*,    }        let arr: [i32; 3] = [1, 2, 3];    display_array(arr);    let arr: [i32; 2] = [1, 2];    display_array(arr); }`Go 的泛型`func MapKeys[K comparable, V any](m map[K]V) []K {    r := make([]K, 0, len(m))    for k := range m {        r = append(r, k)    }    return r } type List[T any] struct {    head, tail *element[T] } type element[T any] struct {    next *element[T]    val  T } func (lst *List[T]) Push(v T) {    if lst.tail == nil {        lst.head = &element[T]{val: v}        lst.tail = lst.head    } else {        lst.tail.next = &element[T]{val: v}        lst.tail = lst.tail.next    } }` |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| *方法中使用泛型* | `*// 方法中使用泛型* impl<T, U> Point<T, U> {    fn x(&self) -> &T {        &self.x    }    fn mixup<V, W>(self, other: Point<V, W>)     -> Point<T, W> {        Point {            x: self.x,            y: self.y,            z: other.z,        }    } } *// 具体的泛型类型实现方法* impl Point<f32, i32> {    fn distance_from_origin(&self) -> f32 {        (self.x.powi(2) + self.y.powi(2)).sqrt()    } }` |                                                              |
| *枚举中使用泛型* | `*// 枚举中使用泛型* enum Option<T> {    *Some*(T),    *None*, }` |                                                              |
| *const 泛型*     | `*// const 泛型* *// T: std::fmt::Debug 代表对类型 T 增加 std::fmt::Debug 特征限制，以此能够打印* fn display_array<T: std::fmt::*Debug*, const N: usize>(arr: [T; N]) {    println!("{:?}", arr); }` |                                                              |

### 特征

- Rust 的**特征 trait 定义了一组可被共享的行为，若实现该特征就能使用这组行为**，与 Go 中的 Interface 接口类似

- Rust 的特征实现与方法类似**需要显式进行声明（而 Go 不需要），同时提供默认实现和继承**

    - | 为类型实现特征（包含特征默认实现） | `pub trait *Summary* {    fn summarize(&self) -> String;    *// 特征方法默认实现*    fn default_summarize(&self) -> String {        String::*from*("(Read more...)")    } } pub struct Post {    pub title: String,    pub author: String, } impl *Summary* for Post {    fn summarize(&self) -> String {        format!("文章{}, 作者是{}", self.title, self.author)    } } pub struct Weibo {    pub username: String,    pub content: String, } impl *Summary* for Weibo {    fn summarize(&self) -> String {        format!("{}发表了微博{}", self.username, self.content)    } }` | `fn main() {    let post = Post { title: "rust".to_string(), author: "rust".to_string() };    let weibo = Weibo { username: "rust".to_string(), content: "rust".to_string() };    let return_trait = returns_summarize();    println!("{}", post.summarize());    println!("{}", weibo.summarize());    println!("{}", return_trait.summarize());    notify(&post);     println!("{}", weibo.name());    println!("{}", weibo.content()); }`Go 中的 interface`type testInterface interface {   TestMethod() } var _ testInterface = (*Test)(nil) type Test struct {   A string } func (t Test) TestMethod() {   panic("implement me") }` |
          | ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
      | 特征作为函数参数                   | `pub fn notify(item: &impl *Summary*) {    println!("Breaking news! {}", item.summarize()); }` |                                                              |
      | 特征作为函数返回值                 | `fn returns_summarize() -> impl *Summary* {    Weibo {        username: String::*from*("return_rust"),        content: String::*from*("return_rust"),    } }` |                                                              |
      | 特征继承                           | `trait *Person* {    fn name(&self) -> String; } *// Person 是 Student 的父 trait* *// 实现 Student 需要也实现 Person* trait *Student*: *Person* {    fn content(&self) -> String; } impl *Person* for Weibo {    fn name(&self) -> String {        format!("{}", self.username)    } } impl *Student* for Weibo {    fn content(&self) -> String {        format!("{}", self.content)    } }` |                                                              |

- Rust 中结构体实现不同 trait 的同名方法时，**不需要强制覆盖（Go 会覆盖），可用 self 或完全限定完成特定调用**

    - | self                             | `pub trait *Summary* {    fn summarize(&self) -> String { String::*from*("summary") } } pub trait *OtherSummary* {    fn summarize(&self) -> String { String::*from*("other summary") } } pub struct Content {} impl *Summary* for Content {} impl *OtherSummary* for Content {} impl Content {    fn summarize(&self) -> String { String::*from*("Content summary") } } fn main() {    let content = Content {};    println!("{}", *Summary*::summarize(&content));    println!("{}", *OtherSummary*::summarize(&content));    println!("{}", content.summarize()); *// 优先调用类型上的方法* }` |
          | -------------------------------- | ------------------------------------------------------------ |
      | 完全限定语法（用于方法没self时） | `pub trait *Summary* {    fn *summarize*() -> String { String::*from*("summary") } } pub struct Content {} impl *Summary* for Content {} impl Content {    fn *summarize*() -> String { String::*from*("Content summary") } } fn main() {    let content = Content {};    // 完全限定语法    println!("{}", <Content as *Summary*>::*summarize*()); }` |

- Rust 中的特征**可利用特征约束**，当特征作为函数参数或返回值时，可限制其参数必须实现约束的特征

   ```Rust
    use std::fmt::{Debug, Display};
    pub trait Summary {
        fn summarize() -> String { String::from("summary") }
    }
    
    // 特征约束
    pub fn notify<T: Summary>(item: &T) { // 等价于 notify(item:&impl Summary)
        println!("Breaking news! {}", item.summarize());
    }
    
    // 多重约束
    pub fn notify_more_bound<T: Summary + Display>(item: &T) { // 等价于 notify_more_bound(item:&(impl Summary + Display))
        println!("Breaking news! {}", item.summarize());
    }
    
    // where约束
    fn some_function<T, U>(t: &T, u: &U) // 等价于 some_function(t:&(impl Display + Clone),u:&(impl Clone + Debug))
        where T: Display + Clone, U: Clone + Debug {}
    
    // 使用特征约束有条件地实现方法或特征
    struct Pair<T> { x: T}
    impl<T: Display + PartialOrd> Pair<T> {}
    ```

- Rust 中的特征使用**关联类型或默认泛型类型参数**语法，来降低当泛型声明造成的可读性

    - | **使用前**                                                   | **使用关联类型后**                                           |
          | ------------------------------------------------------------ | ------------------------------------------------------------ |
      | `trait *Container*<A, B> {    fn contains(&self, a: A, b: B) -> bool; } fn difference<A, B, C>(container: &C)    where C: *Container*<A, B> {}` | `trait *Container*{    type A;    type B;    fn contains(&self, a: &Self::A, b: &Self::B) -> bool; } fn difference<C: *Container*>(container: &C) {}` |
      | **使用前**                                                   | **使用默认泛型类型参数后**                                   |
      | `trait Add<T> {    type Output;    fn add(self, rhs: T) -> Self::Output; } struct Point {} impl *Add*<Point> for Point {    type Output = Point;    fn add(self, rhs: Point) -> Self::Output { todo!() } }` | `trait *Add*<RHS=Self> {    type Output;    fn add(self, rhs: RHS) -> Self::Output; } struct Point {} impl *Add* for Point {    type Output = Point;    fn add(self, rhs: Point) -> Self::Output { todo!() } }` |

> 为解决 Rust 将特征作为函数参数或返回值时，必须使用相同实现特征的类型的问题，后续引入了**特征对象解决**
>
> - 其原理是动态分发的处理，即直到运行时才能确定需要调用什么方法 
> ```Rust
>     trait Draw {
>         fn draw(&self) -> String;
>     }
>     
>     impl Draw for u8 {
>         fn draw(&self) -> String {
>             format!("u8: {}", *self)
>         }
>     }
>     
>     impl Draw for f64 {
>         fn draw(&self) -> String {
>             format!("f64: {}", *self)
>         }
>     }
>     
>     // 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
>     fn draw1(x: Box<dyn Draw>) {
>         // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法
>         x.draw();
>     }
>     
>     fn draw2(x: &dyn Draw) {
>         x.draw();
>     }
>     
>     fn main() {
>         let x = 1.1f64;
>         // do_something(&x);
>         let y = 8u8;
>     
>         // x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
>         // 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上
>         draw1(Box::new(x));
>         // 基于 y 的值创建一个 Box<u8> 类型的智能指针
>         draw1(Box::new(y));
>         draw2(&x);
>         draw2(&y);
>     }
> ```
>
> - 使用具有一定的限制，需要考虑对象安全的特征

### 错误处理

- 对于不可恢复错误（可能严重影响程序运行），Rust 与 Go 类似**引入了 panic 设计**，也可通过`panic!`主动触发

    - 若是 main 线程 panic 则程序会终止，若是其他子线程，该线程会终止

    ```Rust
    fn main() {
        panic!("crash and burn");
    } 
    ```

    ```Go
    func main() {
        panic("crash and burn")
    }
    ```


- 对于可恢复错误，前面枚举提到过，Rust 引入 **Result<T,E> 枚举类型作为返回，对其解构来判断是正确还是错误**

    - 这样可省掉处理异常时的部分性能开销，同时也需要显示处理异常而不能简单的忽略

    ```Rust
    use std::fs::File;
    use std::io::ErrorKind;
    
    fn main() {
        let f = File::open("hello.txt");
        match f {
            Ok(file) => file,
            Err(error) => match error.kind() {
                ErrorKind::NotFound => match File::create("hello.txt") {
                    Ok(fc) => fc,
                    Err(e) => panic!("Problem creating the file: {:?}", e),
                },
                other_error => panic!("Problem opening the file: {:?}", other_error),
            },
        };
    }
    ```

    ```Go
    import (
        "fmt"
        "os"
    )
    
    func main() {
        f, err := os.Open("hello.txt")
        defer f.Close()
        if err != nil {
           if os.IsNotExist(err) {
              f, err = os.Create("hello.txt")
              if err != nil {
                 panic(fmt.Sprintf("Problem creating the file: %v", err))
              }
           } else {
              panic(fmt.Sprintf("Problem opening the file: %v", err))
           }
        }
    }
    ```

- Rust 通过**引入** **`unwrap()`** **、** **`expect()`** 方法简化 panic 传播，同时引入 **`?`** **语法糖来简化错误传播**

    - 相较于 Go 的错误处理（`if err!=nil {....}`）更加简洁

    - `?`的语法糖实际上也能用于 Option 的返回，即为 None 时会向上传播

    - | panic 传播   | `use std::fs::File; fn main() {    let f = File::*open*("hello.txt").unwrap();    let f = File::open("hello.txt").expect("Failed to open hello.txt"); }`等价于 Go 中`func main() {    _, err := os.Open("hello.txt")    if err != nil {       *panic*(nil)    }    _, err = os.Open("hello.txt")    if err != nil {       *panic*("Failed to open hello.txt")    } }` |
          | ------------ | ------------------------------------------------------------ |
      | ? 语法糖传播 | `use std::fs::File; use std::io; use std::io::*Read*; fn main() -> Result<String, io::Error> { *// main 函数也可支持多种返回值*    let mut f = File::*open*("hello.txt")?;    *// 等价于*    *// let mut f = match f {*    *//     Ok(file) => file, // 打开文件成功，将file句柄赋值给f*    *//     Err(e) => return Err(e), // 打开文件失败，将错误返回(向上传播)*    *// };*    let mut s = String::*new*();    f.read_to_string(&mut s)?;    *// Rust 支持链式调用，上述代码等价于*    *// File::open("hello.txt")?.read_to_string(&mut s)?;*    *Ok*(s) } fn first(arr: &[i32]) -> Option<&i32> { // Option 也可使用 ？ 向上传播   let v = arr.get(0)?;   Some(v) }`等价于 Go 中`import (    "os" ) func *ReadUsernameFromFile*() (string, error) {    f, err := os.Open("hello.txt")    defer f.Close()    if err != nil {       return "", err    }    fInfo, _ := f.Stat()    buffer := make([]byte, fInfo.Size())    _, err = f.Read(buffer)    if err != nil {       return "", err    }    return string(buffer), nil }` |

### 生命周期

- Rust 引入生命周期（引用的有效作用域）的设计，就是为了**保证引用安全，避免悬垂引用等安全问题**
- Rust 会检查所有引用的生命周期，**确保被引用的对象存活时间长于引用者**
- 若 Rust 编译器无法推断出引用的生命周期，**就需要通过****`&'`****语法显式标注到值上**
    - 大部分场景不用标注生命周期是因为编译器为简化用户使用，制定了**生命周期消除规则**（这里就不详述了）
    - 静态生命周期用`&' static`标注，表示其生命周期能持续到程序结束，**通常用于字符串字面量和特征对象等**

| 函数标注生命周期                                             | 结构体标注生命周期                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `*// 编译错误，在函数退出时 i 已被释放* fn leak_wrong() -> &i32 {    let i = 5;    &i } *// 编译错误，编译器无法推导出返回引用的生命周期* fn max_wrong(x: &i32, y: &i32) -> &i32 {    if x > y {        &x    } else {        &y    } } *// 编译正确，'a 为生命周期标注，保证x和y至少活得和'a 一样久* *// 最后会取 x、y 生命周期重叠部分* fn max_right<*'a*>(x: &*'a* i32, y: &*'a* i32) -> &*'a* i32 {    if x > y {        &x    } else {        &y    } } fn main() {    let x = 0;    let y = 1;    let max = max_right(&x, &y);    println!("max: {}", max); }` | `*// 结构体标注生命周期* **#[derive(Debug)]** struct ImportantExcerpt<*'a*> {    part: &*'a* str, *// 若有引用类型字段则需要标注生命周期* } fn main() {    *// 编译正确 case*    let novel = String::*from*("Call me Ishmael. Some years ago...");    let i = ImportantExcerpt {        part: novel.as_str(),    };    *// 编译错误 case*    let i;    {        let novel = String::*from*("Call me Ishmael. Some years ago...");        i = ImportantExcerpt {            part: novel.as_str(),        };    }    println!("{:?}", i); }` |

## 学习建议

学习 Rust 的过程好比邓克效应，需要我们保持耐心，花时间去适应 Rust 这门特殊的语言。

![img](https://jih9axn4gg.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDI0NTkxZTQ1ZDQ0YjdhMjAwZmJkZDk0MzkxNzExZGRfNGsySlV5dFJZMUNTSTlSRnJ5Uzc5Z3p0aHJJejlrdlpfVG9rZW46RU5zRGJIWTNib2VWQ2V4TXdKSWNXaEYwbkZmXzE3MDU3NzEyNzk6MTcwNTc3NDg3OV9WNA)

如何修成 Rust 正果，这里参考某位同学给出的学习建议：

- 夯实计算机基础知识
- 学会知识屏蔽
- 学会类比你已有的知识
- 多阅读源码，特别是标准库和优秀开源项目的源码
- 还有最重要的一点：一万小时定律！一定要多写代码！

### 书籍资源

- Rust相关书籍文档
    - https://course.rs/about-book.html（个人觉得比《Rust 权威指南》更适合入门）
- Rust IDE：
    - vscode + rust-analyzer
    - jetbrains [rustover](https://www.jetbrains.com/rust/)