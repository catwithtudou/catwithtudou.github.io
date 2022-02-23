# Go并发编程实战课笔记—读写顺序


# Go并发编程实战课笔记—读写顺序



![image-20210224160420574](https://img.zhengyua.cn/20210224160420.png)



Go官方文档里真闷介绍了Go的内存模型，而这里的内存模型不是指Go对象的内存分配、内存回收和内存整理的规范，而描述的是并发环境中多goroutine读相同变量的时候，变量的可见性条件。

编程语言需要一个规范来明确多线程同时访问同一个变量的可见性和顺序，而这个规范就被叫做内存模型。而这主要目的在于：

- 提供保证，方便在同一个数据同时被多个goroutine访问的情况时可以做一些串行化访问的控制；
- 允许编译器和硬件对程序做一些优化；

## 重排和可见性的问题

**由于指令重排，代码并不一定会按照你写的顺序执行**。

就比如下面这个代码的例子，运行的时候可能会出现半初始化、未初始化的问题等。

```go
var a string
var done bool

func setup(){
  a = "hello"
  done = true
}

func main(){
  go setup()
  for !done{
    
  }
  print(a)
}
```

我们就需要了解`happens-before`概念，其用来描述两个时间的顺序关系，若操作之间能提供`happens-before`关系，那么我们就可以确定保证它们之间的顺序。

## happens-before

**在一个 goroutine 内部，程序的执行顺序和它们的代码指定的顺序是一样的，即使编译器或者 CPU 重排了读写顺序，从行为上来看，也和代码指定的顺序一样**。但是别于另外一个 goroutine 来说，重排就会产生非常大的影响。**因为 Go 只保证 goroutine 内部重排对读写的顺序没有影响**。

- a Happens-before b 定义的顺序为 a -> b

比如我们想要保证对于某一个变量的读操作 r1 绝对能够观察到 写操作 w1，我们就需要同时满足两个条件：

1.  w1 happens before r1；
2. 如果对该变量有其他的写操作 w2，就需要保证 w2->w1->r1或者 w1->r1->w2，绝对不会和 w1、r1 同时发生，或者是在它们之间发生。

**在单个的 goroutine 内部，happens-before 的关系和代码编写的顺序是一致的。**

1. 在 Go 语言中，对变量进行零值的初始化就是一个写操作；
2. 如果对超过机器 word（64bit、32bit 或者其他）大小的值进行读写，那么，就可以看作是对拆成 word 大小的几个读写无序进行；
3. Go 并不提供直接的 CPU 屏障（CPU fence）来提示编译器或者 CPU 保证顺序性，而是使用不同架构的内存屏障指令来实现统一的并发原语。

## Go 语言中保证的 happens-before 关系

### init 函数

main函数一定在导入的包的 init 函数之后执行，且**每个文件最多只能有一个 init 函数**，包下面的多个 init 函数按照它们的**文件名顺序逐个初始化**。

![image-20210224154343784](https://img.zhengyua.cn/20210224154349.png)

### goroutine

**启动 goroutine 的 go 语句的执行，一定 happens before 此 goroutine 内的代码执行**。

根据这个规则可说明如果 go 语句传入的参数是一个函数执行的结果，那这个函数一定先于 goroutine 内部的代码被执行。

```go
var a string

func f(){
  fmt.Println(a)
}

func main(){
  a = "hello"
  go f()
}
```

### Channel

通用的 Channel happens before 关系保证有4条规则：

1. 往 Channel 中的发送操作，happens before 从该 Channel 接收相应数据的动作完成之前，即 nSend -> nReceive。
2. Close 一个 Channel 的调用，肯定 happens before 从关闭的 Channel 中读取出一个零值。

```go
var ch = make(chan struct{},10)
var s string

func f(){
  s = "hello"
  //ch<-struct{}{}
  close(ch)
}

func main(){
  go f()
  <-ch //能够在close之前保证取出零值
  fmt.Println(s)
}
```

3. 对于 unbuffered 的 Channel，读取数据的调用一定 happens before 往此 Channel 发送数据的调用完成。
4. 如果 Channel 的容量是 m（m>0），那么 nReceive -> (n+m)Send。

### Mutex/RWMutex

根据官方描述的 happens-before 关系的保证如下：

对于读写锁 l 的 l.RLock 方法调用，如果存在一个 n，这次的 l.RLock 调用 happens after 第 n 次的 l.Unlock，那么和这个 RLock 相对应的 l.RUnlock 一定 happens before 第 n+1 次 l.Lock。意思是，读写锁的 Lock 必须等待既有的读锁释放后才能获取到。

### WaitGroup

WaitGroup的保证是 **Wait 方法等到计数值归零之后才返回**。

### Once

提供的保证为以下：

**对于 once.Do(f) 调用，f 函数的那个单次调用一定 happens before 任何 once.Do(f) 调用返回**。即函数 f 一定会在 Do 方法返回之前执行。

### atomic

对于 Go 1.15 的官方实现来说，可以保证使用 atomic 的 Load/Store 的变量之间的顺序性。

但现阶段还是不要使用 atomic 来保证顺序性。

