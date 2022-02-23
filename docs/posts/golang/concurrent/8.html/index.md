# Go并发编程实战课笔记—Once


# Go并发编程实战课笔记—Once

> 以下为鸟窝大佬的[Go 并发编程实战课](https://time.geekbang.org/column/intro/100061801) 中摘录的笔记
>
> [代码repo](https://github.com/catwithtudou/golang_concurrent_examples/tree/master/once)



![image-20210108003248240](https://img.zhengyua.cn/img/20210108003255.png)


## Once基本概念

Once的使用方法较为简单，**可以用来执行且仅仅执行一次动作**。

它的使用场景较多应用于**单例对象的初始化或者延迟初始化**等场景。

## Once使用

**Once常常用来初始化单例资源，或者并发访问只需初始化一次的共享资源，或者在测试的时候初始化一次测试资源**。

只暴露了一个方法 Do，可以多次调用 Do 方法，但是只有第一次调用 Do 方法时 f 参数才会执行，这里的 f 是一个无参数无返回值的函数。

```go
func (o *Once) Do(f func())
```

> 标准库中较为常见的比如在标准库内部cache的实现上就使用了Once初始化Cache资源，包括defaultDir的获取，还有一些测试的时候初始化测试的资源。
>
> 其中math/big/sqrt.go种实现的一个数据结构，通过Once封装了一个只初始化一次的值，值得我们学习。
>
> ```go
> // 值是3.0或者0.0的一个数据结构
> var threeOnce struct {
> 	sync.Once
> 	v *Float
> }
> // 返回此数据结构的值，如果还没有初始化为3.0，则初始化
> func three() *Float {
> 	threeOnce.Do(func() { // 使用Once初始化
> 		threeOnce.v = NewFloat(3.0)
> 	})
> 	return threeOnce.v
> }
> ```

## 实现Once

一个正确的Once实现要使用**一个互斥锁**，这样初始化的时候如果有并发的 goroutine，就会进入**doSlow方法**。互斥锁的机制保证只有一goroutine进行初始化，同时利用**双检查的机制（double-checking）**，再次判断o.done 是否为0，如果为0，则是第一次执行，执行完毕后，就将o.done设置为 1，然后释放锁。

```go
type Once struct {
	done uint32
	m sync.Mutex
}
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 {
		o.doSlow(f)
	}
}
func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	// 双检查
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

## 可能出现的错误

### 死锁

无限递归导致的死锁。

```go
var once sync.Once
once.Do(func() {
	once.Do(func() {
		fmt.Println("初始化")
	})
})
```

### 未初始化

如果f方法执行的时候panic，或者f执行初始化资源的时候失败了，这个时候，Once还是会认为初次执行已经成功了，即使再次调用Do方法，也不会再次执行f。

- 可以通过自己实现一个类似Once的并发原语解决

```go

type ReOnce struct {
	m sync.Mutex
	done uint32
}
// 传入的函数f有返回值error，如果初始化失败，需要返回失败的error
// Do方法会把这个error返回给调用者
func (o *ReOnce) Do(f func() error) error {
	if atomic.LoadUint32(&o.done) == 1 { //fast path
		return nil
	}
	return o.slowDo(f)
}
// 如果还没有初始化
func (o *ReOnce) slowDo(f func() error) error {
	o.m.Lock()
	defer o.m.Unlock()
	var err error
	if o.done == 0 { // 双检查，还没有初始化
		err = f()
		if err == nil { // 初始化成功才将标记置为已初始化
			atomic.StoreUint32(&o.done, 1)
		}
	}
	return err
}
```

- 若想要查询初始化过的情况则还需要一个辅助变量来检查

```go
type AnimalStore struct {
	once sync.Once
	inited uint32
}
func (a *AnimalStore) Init(){ // 可以被并发调用
	a.once.Do(func() {
		longOperationSetupDbOpenFilesQueuesEtc()
		atomic.StoreUint32(&a.inited, 1)
	})
}
func (a *AnimalStore) CountOfCats() (int, error) { // 另外一个goroutine
	if atomic.LoadUint32(&a.inited) == 0 { // 初始化后才会执行真正的业务逻辑
		return 0, NotYetInitedError
	}
	//Real operation
}
```

- 但若是有一个Done方法直接使用判断是否初始化过则更为简便

```go
// Once 是一个扩展的sync.Once类型，提供了一个Done方法
type Once struct {
	sync.Once
}
// Done 返回此Once是否执行过
// 如果执行过则返回true
// 如果没有执行过或者正在执行，返回false
func (o *Once) Done() bool {
	return atomic.LoadUint32((*uint32)(unsafe.Pointer(&o.Once))) == 1
}
```

### 重复初始化

若类似于想要实现增加一个Reset方法来重新初始化Once，使之能够重新使用。

- Go核心开发者lan Lance Taylor提供了一个简单的解决方案即reset时使用new方法重新给sync.Once变量遍历重新赋值一个新的实例。

但下面这种情况就会直接panic：

```go
type MuOnce struct {
	sync.RWMutex
	sync.Once
	mtime time.Time
	vals []string
}

// 相当于reset方法，会将m.Once重新复制一个Once
func (m *MuOnce) refresh() {
	m.Lock()
	defer m.Unlock()
	m.Once = sync.Once{}
	m.mtime = time.Now()
	m.vals = []string{m.mtime.String()}
}
// 获取某个初始化的值，如果超过某个时间，会reset Once
func (m *MuOnce) strings() []string {
	now := time.Now()
	m.RLock()
	if now.After(m.mtime) {
		defer m.Do(m.refresh) // 使用refresh函数重新初始化
	}
	vals := m.vals
	m.RUnlock()
	return vals
}

```

我们需要注意从once的实现原理中的lock，若更改了Once的指针则会影响该lock也重新生成即初始化，则defer之后的unlock就会释放未加锁的mutex，所以最后就会报错。
