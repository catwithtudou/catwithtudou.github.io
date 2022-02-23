# Go并发编程实战课笔记—Mutex


# Go并发编程实战课笔记—Mutex

> 以下为鸟窝大佬的[Go 并发编程实战课](https://time.geekbang.org/column/intro/100061801) 中摘录的笔记
>
> [代码repo](https://github.com/catwithtudou/golang_concurrent_examples/tree/master/mutex)


![image-20210103003621340](https://img.zhengyua.cn/img/

同步原语的使用场景：

- 共享资源。并发读写并发资源，会出现数据竞争的问题，所以需要Mutex、RWMutex等并发原语的保护。
- 任务编排。需要goroutine按照一定的规律执行，而goroutine之间有相互等待或者依赖的顺序关系，常常使用WaitGroup或者Channel来实现。
- 消息传递。信息交流以及不同的goroutine之间的线程安全的数据交流，常常使用channel来实现。

**互斥锁Mutex提供两个方法Lock和Unlock：进入临界区之前调用Lock方法，退出临界区的时候调用Unlock方法。**

> 利用`go race detector`检测`data race`的情况。

> 有时候采用嵌入字段的方式，来标注不同作用的mutex。

## 源码解析



![image-20201230095135808](https://img.zhengyua.cn/img/20201230095510.png)

### 初版互斥锁

```go
// CAS操作，当时还没有抽象出atomic包
func cas(val *int32, old, new int32) bool
func semacquire(*int32)
func semrelease(*int32)
// 互斥锁的结构，包含两个字段
type Mutex struct {
key int32 // 锁是否被持有的标识
sema int32 // 信号量专用，用以阻塞/唤醒goroutine
}
// 保证成功在val上增加delta的值
func xadd(val *int32, delta int32) (new int32) {
for {
v := *val
if cas(val, v, v+delta) {
return v + delta
}
}
panic("unreached")
}
// 请求锁
func (m *Mutex) Lock() {
if xadd(&m.key, 1) == 1 { //标识加1，如果等于1，成功获取到锁
return
}
semacquire(&m.sema) // 否则阻塞等待
}
func (m *Mutex) Unlock() {
if xadd(&m.key, -1) == 0 { // 将标识减去1，如果等于0，则没有其它等待者
return
}
semrelease(&m.sema) // 唤醒其它阻塞的goroutine
}
```

![image-20201230095618527](https://img.zhengyua.cn/img/20201230095618.png)

- 由于Mutex 本身并没有包含持有这把锁的goroutine的信息，Unlock也不会对此进行检查，Unlock方法可以被任意的goroutine调用释放锁，即使是没持有这个互斥锁的goroutine也可以。

所以需要注意在使用 Mutex 的时候，必须要保证 goroutine 尽可能不去释放自己未持有的锁，遵循**“谁申请，谁释放”**的原则。

> 在真实的实践中，我们使用互斥锁的时候， 很少在一个方法中单独申请锁，而在另外一个方法中单独释放锁，一般都会在同一个方法中获取锁和释放锁。

- 初版互斥锁有一个问题在于请求锁的goroutine会排队等待获取互斥锁，在高并发场景下不是性能最优的策略，如果能够把锁交给正在占用CPU时间片的goroutine就不用做上下文的切换。

### 新来的goroutine竞争

![image-20201230100300919](https://img.zhengyua.cn/img/20201230100301.png)

```go
func (m *Mutex) Lock() {
// Fast path: 幸运case，能够直接获取到锁
if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
return
}
awoke := false
for {
old := m.state
new := old | mutexLocked // 新状态加锁
if old&mutexLocked != 0 {
new = old + 1<<mutexWaiterShift //等待者数量加一
}
if awoke {
// goroutine是被唤醒的，
// 新状态清除唤醒标志
new &^= mutexWoken
}
if atomic.CompareAndSwapInt32(&m.state, old, new) {//设置新状态
if old&mutexLocked == 0 { // 锁原状态未加锁
break
}
runtime.Semacquire(&m.sema) // 请求信号量
awoke = true
}
}
}
```

- Fast Path：首先通过CAS检测state字段的标志，判断是否持有锁，若没有则直接获得锁；
- 下面循环检查，涉及到state不同标志位的操作，

![image-20201230100846349](https://img.zhengyua.cn/img/20201230100846.png)

```go
func (m *Mutex) Unlock() {
// Fast path: drop lock bit.
new := atomic.AddInt32(&m.state, -mutexLocked) //去掉锁标志
if (new+mutexLocked)&mutexLocked == 0 { //本来就没有加锁
panic("concurrent_programming: unlock of unlocked mutex")
}
old := new
for {
if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken) != 0
return
}
new = (old - 1<<mutexWaiterShift) | mutexWoken // 新状态，准备唤醒goroutine
if atomic.CompareAndSwapInt32(&m.state, old, new) {
runtime.Semrelease(&m.sema)
return
}
old = m.state
}
```

- 若没有锁的情况下进行Unlock则直接panic；
- 若有锁的情况下有两种情况：
    - 若没有waiter，则直接返回；
    - 若有waiter，且没有被唤醒，则需要唤醒一个等待的waiter；

相对于初版的设计，这次改动主要就是**新来的goroutine有机会先获取到锁**。

### 自旋检查锁释放

若新来的或者是被唤醒的goroutine首次获取不到锁则会进行自旋状态尝试检测锁是否被释放。在尝试一定的自旋次数后，再执行原来的逻辑。

```go
func (m *Mutex) Lock() {
// Fast path: 幸运之路，正好获取到锁
if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
return
}
awoke := false
iter := 0
for { // 不管是新来的请求锁的goroutine, 还是被唤醒的goroutine，都不断尝试请求锁
old := m.state // 先保存当前锁的状态
new := old | mutexLocked // 新状态设置加锁标志
if old&mutexLocked != 0 { // 锁还没被释放
if runtime_canSpin(iter) { // 还可以自旋
if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift
atomic.CompareAndSwapInt32(&m.state, old, old|mutexWok
awoke = true
}
runtime_doSpin()
iter++
continue // 自旋，再次尝试请求锁
}
new = old + 1<<mutexWaiterShift
}
if awoke { // 唤醒状态
if new&mutexWoken == 0 {
panic("concurrent_programming: inconsistent mutex state")
}
new &^= mutexWoken // 新状态清除唤醒标记
}
if atomic.CompareAndSwapInt32(&m.state, old, new) {
if old&mutexLocked == 0 { // 旧状态锁已释放，新状态成功持有了锁，直接返回
break
}
runtime_Semacquire(&m.sema) // 阻塞等待
awoke = true // 被唤醒
iter = 0
}
}
```

- 对于临界区代码执行非常短的场景来说这是一个很好的优化，因为临界区的代码耗时较短，锁很快就能释放，而抢夺锁的goroutine不用通过休眠唤醒方式等待调度，直接通过自旋可能就获取到了锁。

### 解决饥饿

上面我们看到为了性能优化将新来的和唤醒的goroutine参与竞争，必然就会引入饥饿的问题。

> Mutex 不能容忍这种事情发生。所以，2016 年 Go 1.9 中 Mutex 增加了饥饿模式，让锁变得更公平，不公平的等待时间限制在1毫秒，并且修复了一个大 Bug：总是把唤醒的 goroutine 放在等待队列的尾部，会导致更加不公平的等待时间。之后，2018 年，Go 开发者将 fast path 和 slow path 拆成独立的方法，以便内联，提高性能。2019 年也有一个 Mutex 的优化，虽然没有 Mutex 做修改，但是，对 Mutex唤醒后持有锁的那个 waiter，调度器可以有更高的优先级去执行，这已经是很细致的性能优化了。

![image-20201230102230186](https://img.zhengyua.cn/img/20201230102230.png)

Mutex让每一个goroutine都有机会获取到锁，而且它也尽可能地让等待较长的 goroutine 更有机会获取到锁。

### 饥饿模式和正常模式

请求锁时调用的 Lock 方法中一开始是 fast path，这是一个幸运的场景，当前的 goroutine 幸运地获得了锁，没有竞争，直接返回，否则就进入了 lockSlow 方法。

> 这种设计，方便编译器对Lock方法进行内联，你也可以在程序开发中应用这个技巧。

- 正常模式下，waiter 都是进入先入先出队列，被唤醒的 waiter 并不会直接持有锁，而是要和新来的 goroutine 进行竞争。新来的 goroutine有先天的优势，它们正在CPU中运行，可能它们的数量还不少，所以在高并发情况下，被唤醒的waiter可能比较悲剧地获取不到锁，这时，它会被插入到队列的前面。如果waite 获取不到锁的时间超过阈值1毫秒，那么，这个Mutex就进入到了饥饿模式。
- 在饥饿模式下，Mutex的拥有者将直接把锁交给队列最前面的 waiter。新来的goroutine不会尝试获取锁，即使看起来锁没有被持有，它也不会去抢，也不会spin，它会乖乖地加入到等待队列的尾部。
- 如果拥有 Mutex 的 waiter 发现下面两种情况的其中之一，它就会把这个 Mutex 转换成正常模式
    - 正常模式拥有更好的性能，因为即使有等待抢锁的 waiter，goroutine 也可以连续多次获取到锁。
    - 饥饿模式是对公平性和性能的一种平衡，它避免了某些 goroutine 长时间的等待锁。在饥饿模式下，优先对待的是那些一直在等待的waiter。

## 常用错误场景

### Lock/Unlock不是成对出现

若Lock/Unlock没有成对出现，则会出现死锁的情况，或者是因为Unlock一个未加锁的Mutex而导致panic。

### Copy已使用的Mutex

Package sync的同步原语在使用后是不能复制的，因为Mutex是一个有状态的对象，它的state字段会记录这个锁的状态。

> Go在运行时会有死锁的检查机制（checkdead()方法），能够发现死锁的goroutine。
>
> vet工具可以检查同步原语。也可以使用第三方库进行检查。

### 重入

> 可重入锁（递归锁）：当一个线程获取锁时，如果没有其它线程拥有这个锁，那么，这个线程就成功获取到这个锁。之后，如果其它线程再请求这个锁，就会处于阻塞等待的状态。但是，如果拥有这把锁的线程再请求这把锁的话，不会阻塞，而是成功返回。
>
> 在通过递归实现一些算法时，调用者不会阻塞或者死锁。

**Mutex不是可重入锁**。Mutex实现没有记录goroutine是否拥有该锁。

若要实现可重入锁，则需要记录持有该锁的goroutine标识。

### 死锁

> 死锁：两个及以上的进程或线程在执行过程中，因争夺共享资源而处于一种互相等待的状态，若无外部干涉则会无法推进下去。

- 可引入第三方的锁来依赖这个锁进行业务处理，避免出现死锁。


