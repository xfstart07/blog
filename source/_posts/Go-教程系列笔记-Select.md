---
title: '[Go 教程系列笔记] Select'
date: 2018-08-01 18:04:33
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

### 什么是 select ？

`select` 语句用于从多个**发送/接收**操作通道中进行选择。select 语句是阻塞的，直到其中一个发送/接收操作准备就绪。如果多个操作都准备好，则随机选择其中一个操作。语法类似 `switch` 除了每个 case 语句都是通道操作。

下面让我们来看一个例子以便更好的理解。

<!-- more -->

```go
func server1(ch chan string) {  
    time.Sleep(6 * time.Second)
    ch <- "from server1"
}
func server2(ch chan string) {  
    time.Sleep(3 * time.Second)
    ch <- "from server2"

}
func main() {  
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```

在上面的例子中，select 语句将阻塞着，直到其中一个 case 准备就绪。在 `server1` 和 `server2` 分别是睡眠 6 和 3 秒。因此 select 将阻塞 3 秒后程序进行打印。

### 实际使用选择

让我们假设我们有一个关键任务应用程序，我们需要尽快将结果返回给用户。该应用程序的数据被复制并存储在世界各地的不同服务器中。假设功能server1和server2实际上与2个这样的服务器通信。每个服务器的响应时间取决于每个服务器的负载和网络延迟。我们将请求发送到两个服务器，然后使用该select语句在相应的通道上等待响应。首先响应的服务器由select选择，其他响应被忽略。这样我们就可以向多个服务器发送相同的请求，并将最快的响应返回给用户:)。

### 默认情况 default

`select` 当其他 case 都没有准备好时，将执行 `default` 语句。这通常用于防止 select 语句阻塞。

```go
func main() {  
    select {
    default:
        // do something
    }
}
```
### Deadlock 和 default case

```go
func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    }
}
```

当 select 语句永远阻塞，没有其他 goroutine 写入此通道，因此将导致死锁。

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:  
main.main()  
    /tmp/sandbox416567824/main.go:6 +0x80
```

如果存在默认情况，则不会发生死锁的情况，因此在没有其他 case 准备就绪时将执行默认情况。

```go
func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    default:
        fmt.Println("default case executed")
    }
}
```

### 随机选择

当 `select` 语句中的多个案例准备就绪时，将随机执行一个 case。

### 空 select

```go
func main() {  
    select {}
}
```

您认为上述计划的输出结果如何？

我们知道select语句将被阻塞，直到其中一个案例被执行。在这种情况下，select语句没有任何情况，因此它将永远阻塞，从而导致死锁。这个程序会因以下输出而感到恐慌:

```go
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select (no cases)]:  
main.main()  
    /tmp/sandbox299546399/main.go:4 +0x20
```