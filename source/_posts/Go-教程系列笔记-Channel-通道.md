---
title: '[Go 教程系列笔记] Channel 通道'
date: 2018-08-01 15:09:45
categories: Go
tags: Go
---

在上一个教程中，我们讨论了如何使用 goroutine 在 Go 中实现并发。在本教程中，我们将讨论有关通道以及 goroutine 如何使用通道进行通信。

### 什么是通道？

通道可以被认为是 goroutine 通信的通道。类似于水在管道中从一端流向另一端的方式，数据可以从一端发送，另一端使用通道接收。

<!-- more -->

### 声明通道

每个通道都有一个与之关联的类型。此类型是允许通道传输的数据类型。不允许使用该通道传输其他类型。

`chan T` 是一种类型为 T 的通道。

通道的零值是 `nil`。`nil` 通道是没有任何用处，因此必须使用 `make` 类似于 `map` 和 `slice` 来定义通道。

让我们写一些声明通道的代码。

```go
func main() {
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```

play: https://play.golang.org/p/oqrK1vDYCsz

`var a chan int` 声明了一个通道，这时通道是零值(nil)的。在 if 条件内的语句定义了通道，这时通道的值是 int 的零值(0)。

像往常一样，简写声明也是定义通道的有效而简洁的方法。

```go
a := make(chan int)
```

### 从通道发送和接收

下面给出了从通道发送和接收数据的语法。

```go
data := <- a // read from channel a
a <- data // write to channel a
```

箭头相对于通道的方向指定是发送或接收数据。

第一行中，箭头从 `a` 指出来，因此我们从通道 `a` 读取值并存储到变量 `data` 中。

第二行中，箭头指向 `a`，因此我们是写入到通道 `a`。

### 默认情况下，发送和接收是阻塞的

默认情况下，对通道的发送和接收是阻塞的，这是什么意思？当数据发送到通道时，发送语句被阻塞着，直到其他 goroutine 从该通道读取。类似地，当从通道读取数据时，读取被阻塞，直到一些 goroutine 将数据写入到该通道。

通道的这种属性有助于 goroutine 有效地进行通信，而无需使用在其他编程语言中非常常见的显式锁或条件变量。

### 通道示例程序

在上一个教程介绍 goroutine 的程序中，我们使用 Sheep 方法让主程序休眠来等待 `go hello()` 的完成，下面我们来用通道重写这个程序。

```go
func hello(done chan bool) {
	fmt.Println("hello world goroutine")
	done <- true
}

func main() {
	done := make(chan bool)
	go hello(done)
	<-done
	fmt.Println("main function")
}
```

在上面的程序中，我们创建了一个布尔值通道 `done`，并将它作为参数传给 `go hello(done)`。然后主程序从 `<-done` 通道接收数据，这行代码是阻塞的，这意味着 goroutine 将数据写入 `done` 通道之前，代码不会移动到下一行。这样就防止了主程序 goroutine 退出。

代码 `<-done` 从完成的 goroutine 接收数据，但不赋值给任何变量中使用或存储该数据。这是完全合法的。

### Deadlock 死锁

使用通道时要考虑的一个重要因素是**死锁**。如果 goroutine 正在通道上发送数据，那么预计其他一些 goroutine 应该接收数据。如果没有发生这种情况，程序将在运行时发生混乱。

```go
func main() {
    ch := make(chan int)
    ch <- 5
}
```

在上面的程序中， `ch` 创建了一个通道，我们将其发送 5 到该通道 `ch <- 5`。在这个程序中，没有其他 goroutine 从通道接收`ch`的数据。因此，从程序在运行时将出现以下错误：

```shell
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
    /tmp/sandbox249677995/main.go:6 +0x80
```

### 单向通道

到目前为止我们讨论的所有通信都是双向通道，即数据可以在它们上发送和接收。也可以创建单向通道，即仅发送或接收数据的通道。

```go
func sendData(sendch chan<- int) {
    sendch <- 10
}

func main() {
    sendch := make(chan<- int)
    go sendData(sendch)
    fmt.Println(<-sendch)
}
```

在上面的程序中，主程序的第一行创建了仅发送的通道。`chan<- int` 表示箭头指向的仅发送通道。我们尝试在仅发送通道接收数据是不允许的。当程序运行时，编译器会报错。

```
invalid operation: <-sendch (receive from send-only type chan<- int)
```

因为我们定义的通道是只能发送，那么就不能接收数据。可以将双向通道转换为仅发送或仅接收通道。

修改为正确的代码如下：

```go
func sendData(sendch chan<- int) {
    sendch <- 10
}

func main() {
    chnl := make(chan int)
    go sendData(chnl)
    fmt.Println(<-chnl)
}
```

### 关闭通道和 for range 通道

发送人能够关闭通道以通知接收者不再在该通道上发送数据。

接收器可以在从通道接收数据时使用附加变量来检查通道是否已经关闭。

```
v, ok := <-ch
```

如果 `ok`是 true，则表示成功从通道的发送操作接收到值。如果 `ok`是 false，则意味着我们正在从关闭的通道读取，从关闭通道读取的值是通道类型的零值。

play：https://play.golang.org/p/XWmUKDA2Ri

`for range` 语法亦可用于从通道接收值，直到它关闭。

让我们使用 `for range` 重写上面的程序。

play：https://play.golang.org/p/JJ3Ida1r_6

重写上一个教程中计算数字的例子：

play：https://play.golang.org/p/oL86W9Ui03

