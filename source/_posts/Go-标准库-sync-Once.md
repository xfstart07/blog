---
title: '[Go 标准库] sync.Once'
date: 2018-08-16 11:13:10
categories: Go
tags: Go
---


`sync.Once` 用来保证 Once 的 Do 只运行一次。

我们来看看 `sync.Once` 的类型结构体：

```
type Once struct {
	m    Mutex
	done uint32
}
```

主要有两个字段，`Mutex` 是一个互斥锁，`done` 是一个整型。

<!-- more -->

### 方法


```go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 1 {
		return
	}
	// Slow-path.
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

`Once` 还有一个方法 `Do(f func())`，这个方法是用来执行只需要执行一次的方法。

首先会使用原子库 `atomic`的方法 `LoadUint32()` 来判断 `done` 是否已经标记为 1，没有标记则表示还没有执行过。

然后使用互斥锁设置临界区，这里还会多判断一步 `done` 是否为 0，防止 `done` 有大于 1 的情况。

最后 `defer` 声明使用原子库 `atomic` 将 `done` 存储为 1，然后执行 `f()`。
