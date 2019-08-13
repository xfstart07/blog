---
title: golang 数据结构 2
date: 2019-08-13 09:24:44
categories: Go
tags:
    - Go
    - 数据结构
---

* channel
* atomic
* sync.Mutex
* sync.WaitGroup
* sync.Pool
* sync.Map

### channel

* [[Go 教程系列笔记] Channel 通道](https://blog.weixinote.com/2018/08/01/Go-%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97%E7%AC%94%E8%AE%B0-Channel-%E9%80%9A%E9%81%93/)
* [[The way to go] 协程间的通信2](mweblib://15180603768126)

基本用法

<!--more-->

```go
var ch chan int 
ch := make(chan int)
ch := make(chan int, 2) // 带缓存 chan

ch <- value // 给 channel 发送值
value := <- ch // 从 channel 接收值

// for range channel
for value := range ch {
}

// 函数中使用
func run(ch chan int){}
func run(ch <-chan int) {} // 指定是一个只能接收 channel 
func run(ch chan<- int) {} // 指定是一个只能发送 channel 

close(ch) // 关闭 channel，关闭后不能再往 channel 发送消息
```

### atomic

https://golang.org/pkg/sync/atomic/

各种原子操作

```go
var data int32
data = atomic.AddInt32(data, 1) // int类型原子操作，还有方法：AddInt64，AddUint32 etc.

// 支持任何类型
var value atomic.Value

value.Store(data) // 存储
data := value.Load().(int32) // 获取

// 进行 compare-and-swap(CAS) 操作
swaped := atomic.CompareAndSwapInt32(&data, data, 2)
```

atomic.Value 可以使用在项目中存储 config。

[比较并交换 CAS](https://zh.wikipedia.org/wiki/%E6%AF%94%E8%BE%83%E5%B9%B6%E4%BA%A4%E6%8D%A2)

### sync.Mutex

[[Go 教程系列笔记] Mutex（互斥锁）](https://blog.weixinote.com/2018/08/01/Go-%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97%E7%AC%94%E8%AE%B0-Mutex%EF%BC%88%E4%BA%92%E6%96%A5%E9%94%81%EF%BC%89/)

```go
// 互斥锁一般配合 struct 一起用
clazz := struct {
    data int
    sync.Mutex
}

clazz.Lock() // 申请锁
clazz.data++
clazz.Unlock() // 释放锁
```

#### sync.RWMutex

读写锁是可以获得任意多的读锁和一个写锁。

```go
// 互斥锁一般配合 struct 一起用
clazz := struct {
    data int
    sync.Mutex
}

clazz.RLock() // 申请读锁
fmt.Println(clazz.data)
clazz.RUnlock() // 释放读锁

clazz.Lock() // 申请写锁
clazz.data++
clazz.Unlock() // 释放写锁
```

> 在获取写锁时，必须读锁和写锁都必须释放后才能获得。不然会进入阻塞。

### sync.Pool

Pool 是一组可以单独保存和获取的临时对象。

**任何存储在 Pool 的元素可能在任何时候被删除而不会通知。**

Pool 在多个 goroutines 同时使用是线程安全的。

Pool 目的是缓存已分配但未使用的元素。以减少内存分配，缓解 GC 的压力。

基本使用

```go
var bufPool = sync.Pool{
	New: func() interface{} {
		return new(bytes.Buffer)
	},
}

func Log(w io.Writer, key, val string) {
    b := bufPool.Get().(*bytes.Buffer)
    b.Reset() // Pool 获取的元素都需要做初始化
    
    b.WriteString(time.Now().Format(time.RFC3339))
    b.WriteByte(' ')
    b.WriteString(key)
    b.WriteByte('=')
    b.WriteString(val)
    w.Write(b.Bytes())
    bufPool.Put(b) // 放回 Pool
}

// Pool 定义
type Pool struct {
    // New 设置一个函数用来创建元素
    New func() interface{}
}

// Pool 方法
func (p *Pool) Get() interface{} // 获取元素
func (p *Pool) Put(x interface{}) // 放回元素
```

> sync.Pool 的问题是，Pool 中的对象会随时被 GC 清理掉，这使得 sync.Pool 只适合做简单的对象池，而不适合做连接池(http.Client，redis.Client 这类连接对象)。

https://studygolang.com/articles/16282

### sync.Map

https://golang.org/pkg/sync/#Map

使用 sync.Map 的原因是 golang 的 map 的读写不是并发安全的。

Map 是像 `map[interface{}]interface{}` 但在多个 goroutines 使用是并发安全。

Map 类型优化的两个点：

* (1) 当一个键值对只写一次，但读很多次，就像在只增长的缓存中一样。
* (2) 当多个 goroutines 读取、写入和覆盖不相交的键值对时。

当是这两类情况下，比起 map 搭配 Mutex 或 RWMutex，使用 Map 可以显著减少锁的争用情况。

基本使用

```go
var hash sync.Map

hash.Store("name", "leon") // 存储 key, value
value, ok := hash.Load("name") // 读取

actual, loaded := hash.LoadOrStore("name", "leonXu") // 读取或更新

hash.Delete("name") // 删除

final = make(map[interface{}]interface{})
// 循环 Map
hash.Range(func(k, v interface{}) bool {
	final[k] = v
	return true
})
```

[Go 1.9 sync.Map揭秘](https://colobu.com/2017/07/11/dive-into-sync-Map/)

## Ref

* https://golang.org/pkg/sync
* https://golang.org/pkg/sync/atomic/