---
title: '[Go 教程系列笔记] Mutex（互斥锁）'
date: 2018-08-01 18:06:37
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

在本教程中，我们将了解互斥锁。我们还将学习如何使用互斥锁和通道解决竞争问题。

### 临界区

在讲互斥锁之前，了解并发编程中**临界区**的概念非常重要。当程序同时运行时，多个 goroutine 同时访问修改共享资源，修改共享资源的这段代码称为**临界区**。例如，假设我们要将变量 x 递增1.

```
x = x + 1
```

只要上面的代码被一个 goroutine 访问，就不会有任何问题。

<!-- more -->

让我们看看为什么当有多个 goroutine 同时运行时，这段代码会失败。为简单起见，我们假设有2个 goroutine 同时运行上面的代码行。

在内部，上面的代码行将由系统按下面的步骤执行。

1. 获取 x 的当前值
2. 计算 x+1 
3. 将步骤2中计算的值分配给 x

当这三个步骤仅由一个 goroutine 进行时，一切都很顺利。

让我们讨论当2个 goroutine 同时运行此代码会发生什么。下图描绘了两个 goroutine 同时访问代码行时可能发生的情况。

![cs5](http://blogcdn.weixinote.com/2019-08-14-cs5.png)


图中，第一步协程1当前x值是0，计算`x+1`，然后系统切换上下文到协程2，第二步，协程2当前x值是0，计算`x+1`，这时系统又切换上下文到协程1，进行分配x值，然后又切换上下文到协程2，进行分配x值，最后，x的值还是1.

让我们再看看可能发生的不同情况：

![cs-6](http://blogcdn.weixinote.com/2019-08-14-cs-6.png)

在上面的场景中，协程1开始执行并完成3个步骤，这时x值是1，然后开始执行协程2，现在x的值已经是1了，在协程2执行完成，x的值就是2了。

因此，在这两种情况下，你可以看到 x 的最终值是1或2取决于上下文切换的方式。这种类型的不良情况，其中程序的输出取决于 goroutine 的执行顺序，称为**竞争条件**。

为了避免竞争条件，可以通过使用 Mutex 实现。

### Mutex 互斥

Mutex 用于提供锁定机制，以确保在任何时间点只有一个 goroutine 在临界区运行，已防止发生竞争条件。

`sync` 包中提供了 Mutex。Mutex 定义了两个方法，即 `Lock` 和 `Unlock`，在 `Lock`和`Unlock` 之间将仅由一个 goroutine 被执行，从而避免了竞争条件。

```
mutex.Lock()  
x = x + 1  
mutex.Unlock()
```

在上面的代码中，`x=x+1`将在任何时间点仅由一个 goroutine 执行，从而防止竞争条件。

**如果一个 goroutine 已经 `Lock`，如果一个新的 goroutine 试图 `Lock`，新的 goroutine 将会阻塞，直到 Mutex Unlock.**

### 有竞争条件的程序

我们将编写一个具有竞争条件的程序，在接下来的部分中我们将修复竞争条件。

```go
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup) {  
    x = x + 1
    wg.Done()
}
func main() {  
    var w sync.WaitGroup
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

请在本地运行此程序，因为操作是确定性的，操作上不会出现比赛条件。在本地计算机上多次运行此程序，您可以看到由于竞争条件，每次输出都会有所不同。其中一些我所遇到的产出是`final value of x 941`，`final value of x 928`，`final value of x 922`等。

### 使用互斥锁解决竞争条件

在上面的程序中，我们产生了1000个Goroutines。如果每个都将x的值递增1，则x的最终期望值应为1000.在本节中，我们将使用互斥锁修复上述程序中的竞争条件。

```go
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, m *sync.Mutex) {  
    m.Lock()
    x = x + 1
    m.Unlock()
    wg.Done()   
}
func main() {  
    var w sync.WaitGroup
    var m sync.Mutex
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, &m)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

**Mutex**是一个结构类型，在第 15 行我们创建了一个零值的变量m的Mutex类型。在上面的程序中，我们更改了`increment`函数，以便增加x的代码`x = x + 1`在`m.Lock()`和`m.Unlock()` 之间。现在这段代码没有任何竞争条件，因为在任何时候只允许一个Goroutine执行这段代码。

### 使用 channel 解决竞争条件

我们也可以使用通道解决竞争条件。让我们看看如何实现的。

```go
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, ch chan bool) {  
    ch <- true
    x = x + 1
    <- ch
    wg.Done()   
}
func main() {  
    var w sync.WaitGroup
    ch := make(chan bool, 1)
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, ch)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

在上面的程序中，我们创建了一个缓冲容量1的通道，并将其传递给`increment`的Goroutine。此缓冲通道用于确保只有一个Goroutine访问增加x的代码的关键部分。这是通过传递true到第 8行号中的缓冲通道来完成的，然后 x增加。由于缓冲通道的容量为1，所有其他尝试写入此通道的Goroutines都会被阻塞，直到在第9行增加x后从该通道读取该值。实际上，这只允许一个Goroutine访问临界区。

这个程序也打印

```
final value of x 1000  
```

### Mutex vs Channel

我们使用互斥锁和通道解决了竞争条件问题。那么我们如何决定何时使用呢？答案在于你要解决的问题。如果你解决的问题更合适互斥锁，那么请继续使用互斥锁。如果需要，请不要犹豫使用互斥锁。如果问题更适合通道，那么使用它：）（没有银弹）

大多数 Go 新手尝试使用通道解决每个并发问题，因为它是该语言的一个很酷的功能。这是错误的，语言为我们提供了使用 Mutex 和 Channel 的选择，并且选择任何一种都没有错。

一般情况下，当 goroutine 需要互相通信时使用通道，当只有一个 goroutine 应该访问代码的临界区时使用互斥。

在我们上面的问题情况下，我宁愿使用互斥锁，因为这个问题不需要 goroutine 之间任何通信。因此互斥锁是一种自然的选择。

我的建议是根据问题选择工具，不要试图让问题适应工具。

