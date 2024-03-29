---
title: '[Go 教程系列笔记] goroutine（协程）'
date: 2018-07-31 13:49:02
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

在上一章中，我们讨论了并发性以及它与并行性的不同之处。在本章中，我们将讨论如何使用 goroutine 在 Go中实现并发。

### 什么是 goroutine？

goroutine 可以被认为是轻量级线程。与线程相比，创建 goroutine 的成本很小。因此，Go 应用程序通常会同时运行数千个 goroutine。

goroutine 也被称为**协程**，**协程的函数或方法与其他函数或方法是同时运行。**

<!-- more -->

### goroutine 优于线程的优点

* 与线程相比，goroutine 非常便宜，它们的堆栈大小只有几KB，堆栈可能根据应用程序的需要增长和缩小，而在线程的情况下，堆栈大小必须指定并且固定的。
* goroutine 被**多路复用**到较少数量的 OS(系统) 线程。程序中可能只有一个线程但有数千个 goroutine。如果该线程中的任何 goroutine 阻塞了，则会创建另一个 OS 线程，并将剩余的 goroutine 移动到新的 OS 线程。所有这些都由运行时（runtime）处理，我们作为程序员从这些复杂的细节中抽象出来，并给出一个干净的 API 来处理并发。
* goroutine 使用 Channel 进行通信。设计 Channel 可以防止 goroutine 访问共享内存时发生竞争。Channel 可以被认为是 goroutine 通信的管道。我们将在下一个教程中详细讨论 Channel。

### 如何使用 goroutine？

使用关键字为函数或方法调用添加前缀 `go`，将同时运行新的 goroutine.

```go
func hello() {  
    fmt.Println("Hello world goroutine")
}
func main() {  
    go hello()
    fmt.Println("main function")
}
```

在上面 `go hello()` 创建了新的 goroutine，现在 `hello()` 函数将与 `main()` 函数同时运行。`main()` 也是用一个 goroutine 运行的，称为 `main goroutine`.

但是上面代码的运行中，只输出了 `main function` 文本。为什么 `hello()` goroutine 没有运行呢？我们需要了解 go 程序的两个主要特性，以了解为什么会发生这种情况。

* 当一个新的 goroutine 启动时，goroutine 启动后立即返回，与函数不同，程序不会等 goroutine 执行完成。在 goroutine 调用之后，程序会立即返回到下一行代码，并忽略 goroutine 的任何返回值。
* `main goroutine` 会运行任何其他 goroutine，如果 main goroutine 终止，则该程序将被终止，并且其他 goroutine 将得不到运行机会。

现在我们理解为什么 `hello()` 没有执行了，在调用了 `go hello()` 之后，程序没有等待 `hello()` 执行完，而是接着执行打印 `main function`，然后 `main goroutine` 终止，因此 `hello()` 的 goroutine 没有机会执行。 

让我们现在来解决这个问题。

```go
func hello() {  
    fmt.Println("Hello world goroutine")
}
func main() {  
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}
```

在上面我们调用了 time 包的 Sleep 方法，这个方法是让正在执行它的 goroutine 睡眠，在这种情况下，`main goroutine` 进入睡眠状态1秒钟。现在在主程序终止之前有足够的时间执行了，`hello()` 打印了 `Hello world goroutine`，然后主程序等待1秒钟之后打印，程序终止。

> 这种在主程序中使用 Sleep 等待其他 goroutine 执行完成的方式是我们用来理解 goroutine 是如何工作的，这种方式是很 hack 。channel 可用于阻塞主程序，直到其他goroutine 执行完成。

### 启动多个 goroutine

让我们再写一个程序，启动多个 goroutine 以更好地理解 goroutine。

```go
func numbers() {  
    for i := 1; i <= 5; i++ {
        time.Sleep(250 * time.Millisecond)
        fmt.Printf("%d ", i)
    }
}
func alphabets() {  
    for i := 'a'; i <= 'e'; i++ {
        time.Sleep(400 * time.Millisecond)
        fmt.Printf("%c ", i)
    }
}
func main() {  
    go numbers()
    go alphabets()
    time.Sleep(3000 * time.Millisecond)
    fmt.Println("main terminated")
}
```

上面的程序开始启用两个 goroutine 排队，两个 goroutine 同时运行。`numbers` 协程每休眠250毫秒打印 `i`，一直打印到 5. 类似的 `alphabets` 休眠 400 毫秒打印一个字符。主程序休眠 3000毫秒之后终止。

下面描绘了该程序的工作原理：

![Goroutines-explained](http://blogcdn.weixinote.com/2019-08-14-Goroutines-explained.png)


每个框代表一个 goroutine 的运行时间。图像是自我解释的画的，让你能够理解程序的工作原理。

**注1：堆栈是什么？**  

> 维基：栈（英语：stack）又称为栈或堆叠，是计算机科学中一种特殊的串列形式的抽象数据类型，其特殊之处在于只能允许在链表或数组的一端（称为堆栈顶端指针，英语：top）进行加入数据（英语：push）和输出数据（英语：pop）的运算。另外栈也可以用一维数组或链表的形式来完成。堆栈的另外一个相对的操作方式称为队列。

**注2：多路复用是什么意思？**

> 维基：多路复用（Multiplexing，又称“多工”）是一个通信和计算机网络领域的专业术语，在没有歧义的情况下，“多路复用”也可被称为“复用”。多路复用通常表示在一个信道上传输多路信号或数据流的过程和技术。因为多路复用能够将多个低速信道整合到一个高速信道进行传输，从而有效地利用了高速信道。通过使用多路复用，通信运营商可以避免维护多条线路，从而有效地节约运营成本。

![](http://blogcdn.weixinote.com/2019-08-14-15326620055027.jpg)