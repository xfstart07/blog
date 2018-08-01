---
title: '[Go 教程系列笔记] 缓冲通道和工作池'
date: 2018-08-01 15:19:31
categories: Go
tags: Go
---

我们在上一个教程中讨论的所有通道基本上都是无缓冲的。正如我们在通道教程中详细讨论的那样，发送和接收无缓冲的通道都是阻塞的。

可以使用缓冲区创建通道。仅当缓冲区已满时才会阻塞对缓冲通道的发送。类似地，仅当缓冲区为空才阻塞从缓冲通道接收。

可以通过将额外的容量参数传递给 `make` 指定缓冲区大小的函数来创建缓冲通道。

```
ch := make(chan type, capacity)
```

对于具有缓冲区的通道，上述语法中的容量应大于0。默认情况下，无缓冲通道的容量为0。

<!-- more -->

play: https://play.golang.org/p/It-em11etK

### 另一个例子

play：https://play.golang.org/p/bKe5GdgMK9

代码中的方法 `close(ch)` 是将通道关闭（写），这样在 `for range` 会收到结束的信息。

### Deadlock 死锁

```go
func main() {
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    ch <- "steve"
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

在上面的程序中，`ch` 的写入容量是2的缓冲通道。当写入第三个时，由于通道已超过其容量，因此写入被阻塞，现在一些 goroutine 必须从通道读取以便继续写入，但在这种情况下，没有从该通道同时读取。因此会出现**死锁**，程序会在运行时出现一下信息：

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
    /tmp/sandbox274756028/main.go:11 +0x100
```

### 长度与容量

缓冲通道的容量是通道可以容纳的值的数量。这是我们使用 `make` 函数创建缓冲通道时指定的值。

缓冲通道的长度是当前在其中存在的元素数。

```
func main() {
    ch := make(chan string, 3)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println("capacity is", cap(ch)) //3
    fmt.Println("length is", len(ch))  //2
    fmt.Println("read value", <-ch)
    fmt.Println("new length is", len(ch)) //1
}
```

### WaitGroup

本教程的下一部分是关于 Worker Pools。要了解工作池，我们首先需要了解`WaitGroup`它在工作池的使用。

play：https://play.golang.org/p/CZNtu8ktQh

`WaitGroup` 是一种结构类型。`WaitGroup` 有三个方法`Add`，`Done`，`Wait`，分别是增加一个，完成一个，等待全部完成。

### Worker Pool 实现

缓冲通道的一个重要用途是实现工作池。

通常，工作池是一组线程，它们正在等待分配给它们的任务。一旦完成分配的任务，它们就会再次为下一个任务提供服务。

以下是工作池的核心功能：

* 创建一个 goroutine 池，它监听输入缓冲的通道，等待分配工作
* 向输入缓冲通道添加工作
* 工作完成后，将结果写入输出缓冲通道
* 从输出缓冲通道读取和打印结果

play: https://play.golang.org/p/au5islUIbx