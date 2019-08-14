---
title: '[Go 教程系列笔记]Interface 第一部分'
date: 2018-07-25 23:22:31
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

### 什么是接口？

面向对象世界中的接口一般定义是“**接口定义对象的行为**”。它只指定对象应该做什么？实现此行为（实现细节）的方法取决于对象。

在 Go 中，接口是一组方法签名。当一个类型为接口中的所有方法提供了定义时，就说它实现了接口。它和 OOP 世界非常相似。接口指定类型应具有的方法，类型决定如何实现这些方法。

例如：`WashingMachine` 可以是具有方法签名 `Cleaning()` 和 `Drying()` 的接口。任何为 `Cleaning` 和 `Drying` 提供定义的类型都可以说实现了 `WashingMachine`。

<!-- more -->

### 声明并实现接口

让我们创建一个接口并实现它的程序。

```go
package main

import (  
    "fmt"
)

//interface definition
type VowelsFinder interface {  
    FindVowels() []rune
}

type MyString string

//MyString implements VowelsFinder
func (ms MyString) FindVowels() []rune {  
    var vowels []rune
    for _, rune := range ms {
        if rune == 'a' || rune == 'e' || rune == 'i' || rune == 'o' || rune == 'u' {
            vowels = append(vowels, rune)
        }
    }
    return vowels
}

func main() {  
    name := MyString("Sam Anderson")
    var v VowelsFinder
    v = name // possible since MyString implements VowelsFinder
    fmt.Printf("Vowels are %c", v.FindVowels())

}
```

在上面创建了一个名为 `VowelsFinder` 的接口类型，它有一个方法签名 `FindVowels() []rune`。

然后创建一个类型 `MyString`。

我们将方法 `FindVowels() []rune` 添加到接收器类型 `MyString` 。现在 `MyString` 实现了接口 `VowelsFinder`。这和 Java 等其他语言不通，其中类必须明确声明使用 `implements` 关键字实现接口。但是在 Go 中不需要这样，在 Go 中是隐式实现的，如果类型包含接口中声明的所有方法，那么就表示实现了这个接口。

### 实际使用接口

我们将编写一个简单的程序，根据员工的个人工资计算公司的总支出。为简洁起见，我们假设所有费用均以美元计算。

假设我们公司有两种雇员，`Permanent` 和 `Contract`。

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    CalculateSalary() int
}

type Permanent struct {  
    empId    int
    basicpay int
    pf       int
}

type Contract struct {  
    empId  int
    basicpay int
}

//salary of permanent employee is sum of basic pay and pf
func (p Permanent) CalculateSalary() int {  
    return p.basicpay + p.pf
}

//salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {  
    return c.basicpay
}

/*
total expense is calculated by iterating though the SalaryCalculator slice and summing  
the salaries of the individual employees  
*/
func totalExpense(s []SalaryCalculator) {  
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {  
    pemp1 := Permanent{1, 5000, 20}
    pemp2 := Permanent{2, 6000, 30}
    cemp1 := Contract{3, 3000}
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)

}
```

这样做的最大优点是`totalExpense`可以扩展到任何新员工类型而无需更改任何代码。

### 接口内部表示

可以认为接口由元组内部表示（type, value）,`type` 是接口的基础具体类型，`value` 保存具体类型的值。

让我们写一个程序来更好地理解。

```go
package main

import (  
    "fmt"
)

type Tester interface {  
    Test()
}

type MyFloat float64

func (m MyFloat) Test() {  
    fmt.Println(m)
}

func describe(t Tester) {  
    fmt.Printf("Interface type %T value %v\n", t, t)
}

func main() {  
    var t Tester
    f := MyFloat(89.7)
    t = f
    describe(t)
    t.Test()
}
```

### 空接口

具有零方法的接口称为空接口。它表示为 `interface{}`。由于空接口的方法为零，因此所有类型都实现了空接口。

```go
package main

import (  
    "fmt"
)

func describe(i interface{}) {  
    fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {  
    s := "Hello World"
    describe(s)
    i := 55
    describe(i)
    strt := struct {
        name string
    }{
        name: "Naveen R",
    }
    describe(strt)
}
```

该`describe(i interface{})`函数将空接口作为参数，因此可以传递任何类型。

### 类型断言

类型断言是用于提取接口的基础值(underlying value)。

`i.(T)` 语法是用于获取 `i` 接口的具体类型的基础值。

```go
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    // 从 i 获得底层类型的值
    s := i.(int) //get the underlying int value from i
    fmt.Println(s)
}
func main() {  
    var s interface{} = 56
    assert(s)
}
```

如果程序中的具体类型不是 int，会发生什么？

如果上面的程序中，我们将具体类型为 string 的传递给尝试断言来提取 int 值的函数。
程序会发生恐慌，提示错误：`panic: interface conversion: interface {} is string, not int`。

要解决上述问题，我们可以使用语法：

```
v, ok := i.(T)  
```

如果 ok 为真，则类型断言正确，如果 ok 为假，并且 v 为具体类型的零值，而且**程序不会发生恐慌**。

### Type Switch

`type switch` 是用于将接口的具体类型和各个 case 语句中指定的类型进行比较。类似于 `switch case`，唯一的区别是 `type switch` 指定的是类型而不是 `switch case` 中的值。

```go
switch i.(type) {
case string:
    fmt.Printf("I am a string and my value is %s\n", i.(string))
case int:
    fmt.Printf("I am an int and my value is %d\n", i.(int))
default:
    fmt.Printf("Unknown type\n")
}
```

**还可以将类型和接口进行比较。如果我们有一个类型，并且该类型实现了一个接口，则可以将此类型与它实现的接口进行比较。**

让我们写一个程序，以便更清晰。

```go
package main

import "fmt"

type Describer interface {  
    Describe()
}
type Person struct {  
    name string
    age  int
}

func (p Person) Describe() {  
    fmt.Printf("%s is %d years old", p.name, p.age)
}

func findType(i interface{}) {  
    switch v := i.(type) {
    case Describer:
        v.Describe()
    default:
        fmt.Printf("unknown type\n")
    }
}

func main() {  
    findType("Naveen")
    p := Person{
        name: "Naveen R",
        age:  25,
    }
    findType(p)
}
```

### 资源

https://golangbot.com/interfaces-part-1/