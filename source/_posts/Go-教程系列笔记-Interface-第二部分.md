---
title: '[Go 教程系列笔记]Interface 第二部分'
date: 2018-07-26 22:29:47
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

### 使用指针接收器和值接收器实现接口

我们在第1部分中讨论的所有示例接口都是使用值接收器实现的。也可以使用指针接收器实现接口。在使用指针接收器实现接口时需要注意一些细微之处。让我们了解使用以下程序。

```go
type Describer interface {
	Describe()
}

type Person struct {
	name string
	age int
}

func (p Person) Describe() { // value receiver
	fmt.Printf("%s is %d years old\n", p.name, p.age)
}

type Address struct {
	state string
	country string
}

func (a *Address) Describe() { // pointer receiver
	fmt.Printf("State %s Country %s", a.state, a.country)
}

func main() {
	var d1 Describer
	p1 := Person{"Sam", 25}
	d1 = p1
	d1.Describe()

	p2 := Person{"James", 32}
	d1 = &p2
	d1.Describe()

	////----------------

	var d2 Describer
	a := Address{"Washington", "USA"}

	// Cannot use a (type Address) as type Describe in assignment
	//d2 = a
  // 在赋值时不能使用（Address 类型）作为 Describe 类型，Address 没有实现 Describe

	d2 = &a

	d2.Describe()
}
```

<!-- more -->

* 第一部分总结：**带值接受器的方法同时接受指针和值类型的调用**
* 第二部分总结：**原因是在任何是指针或能获取其地址的实例上调用指针值方法是合法的。存储在接口中的具体值是不可寻址的，因此编译器不可能自动获取地址。**

### 实现多个接口

一个类型可以实现多个接口。

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var s SalaryCalculator = e
    s.DisplaySalary()
    var l LeaveCalculator = e
    fmt.Println("\nLeaves left =", l.CalculateLeavesLeft())
}
```

### 嵌入接口

尽管 Go 不提供继承，但可以通过嵌入其他接口来创建新接口。

```go
type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type EmployeeOperations interface {  
    SalaryCalculator
    LeaveCalculator
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var empOp EmployeeOperations = e
    empOp.DisplaySalary()
    fmt.Println("\nLeaves left =", empOp.CalculateLeavesLeft())
}
```

### 接口零值

接口的零值是 `nil`。`nil` 是接口的底层值，也是具体类型。

```go
package main

import "fmt"

type Describer interface {  
    Describe()
}

func main() {  
    var d1 Describer
    if d1 == nil {
        fmt.Printf("d1 is nil and has type %T value %v\n", d1, d1)
    }
}
```

如果我们尝试在 `nil` 接口上调用方法，程序将会发生错误(panic)。因为`nil`接口既没有底层值也没有具体类型。

运行时报错误：无效的内存地址或nil指针取消引用
```
[signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xc8527]"
```