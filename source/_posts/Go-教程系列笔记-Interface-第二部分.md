---
title: '[Go 教程系列笔记]Interface 第二部分'
date: 2018-07-26 22:29:47
categories: Go
tags: Go
---

### 使用指针接收器和值接收器实现接口


```
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

第一部分总结：带值接受器的方法同时接受指针和值类型的调用

<!-- more -->

第二部分总结：原因是任何是指针或能获取其地址的实例上调用指针值方法是合法的。存储在接口中的具体值是不可寻址的，因此编译器不可能自动获取地址。

### 实现多个接口

一个类型可以实现多个接口。

play：https://play.golang.org/p/DJxS5zxBcV

### 嵌入接口

尽管 Go 不提供继承，但可以通过嵌入其他接口来创建新接口。

```
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
```

play: https://play.golang.org/p/Hia7D-WbZp

### 接口零值

接口的零值是 `nil`。`nil` 是接口的底层值，也是具体类型。

play：https://play.golang.org/p/vwYHC6Y78H

如果我们尝试在 `nil` 接口上调用方法，程序将会发生错误(panic)。因为`nil`接口既没有底层值也没有具体类型。

```
type Describer interface {
    Describe()
}

func main() {
    var d1 Describer
    d1.Describe() // panic
}
```