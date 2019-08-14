---
title: '[Go 教程系列笔记] 缓冲通道和工作池'
date: 2018-08-01 15:19:31
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

我们在上一个教程中讨论的所有通道基本上都是无缓冲的。正如我们在通道教程中详细讨论的那样，发送和接收无缓冲的通道都是阻塞的。

可以创建带缓冲区的通道。仅当缓冲区已满时才会阻塞对缓冲通道的发送。类似地，仅当缓冲区为空才阻塞从缓冲通道接收。

可以通过将额外的容量参数传递给 `make` 指定缓冲区大小的函数来创建缓冲通道。

```go
ch := make(chan type, capacity)
```

对于具有缓冲区的通道，上述语法中的容量应大于0。默认情况下，无缓冲通道的容量为0。

<!-- more -->

让我们写一些代码并创建一个缓冲的通道。

```go
package main

import (  
    "fmt"
)


func main() {  
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println(<- ch)
    fmt.Println(<- ch)
}
```

### 另一个例子

```go
package main

import (  
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)

    }
}
```

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

在上面的程序中，`ch` 的写入容量是2的缓冲通道。当写入第三个时，由于通道已超过其容量，因此写入被阻塞，现在一些 goroutine 必须从通道读取以便继续写入，但在这种情况下，没有从该通道同时读取。因此会出现**死锁**，程序会在运行时出现以下信息：

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:  
main.main()  
    /tmp/sandbox274756028/main.go:11 +0x100
```

### 长度与容量

缓冲通道的容量是通道可以容纳的值的数量。这是我们使用 `make` 函数创建缓冲通道时指定的值。

缓冲通道的长度是当前在其中存在的元素数。

```go
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

让我们停止理论并立即编写一些代码😀

```go
package main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

`WaitGroup` 是一种结构类型。`WaitGroup` 有三个方法`Add`，`Done`，`Wait`，分别是增加一个，完成一个，等待全部完成。

### Worker Pool 实现

缓冲通道的一个重要用途是实现工作池。

通常，工作池是一组线程，它们正在等待分配给它们的任务。一旦完成分配的任务，它们就会再次为下一个任务提供服务。

以下是工作池的核心功能：

* 创建一个 goroutine 池，它监听输入缓冲的通道，等待分配工作
* 向输入缓冲通道添加工作
* 工作完成后，将结果写入输出缓冲通道
* 从输出缓冲通道读取和打印结果

完整的程序：

```go
package main

import (  
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {  
    id       int
    randomno int
}
type Result struct {  
    job         Job
    sumofdigits int
}

var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)

func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
func main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```
