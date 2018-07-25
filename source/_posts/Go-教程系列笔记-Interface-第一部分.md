---
title: '[Go 教程系列笔记]Interface 第一部分'
date: 2018-07-25 23:22:31
categories: Go
tags: Go
---

### 什么是接口？

面向对象世界中的接口一般定义为“**接口定义对象的行为**”。它只指定对象应该做什么？实现此行为（实现细节）的方法取决于对象。

在 Go 中，接口是一组方法签名。当一个类型为接口中的所有方法提供了定义时，就说它实现了接口。它和 OOP 世界非常相似。接口指定类型应具有的方法，类型决定如何实现这些方法。

例如：`WashingMachine` 可以是具有方法签名 `Cleaning()` 和 `Drying()` 的接口。任何为 `Cleaning` 和 `Drying` 提供定义的类型都可以说实现了 `WashingMachine`。

<!-- more -->

### 声明并实现接口

让我们创建一个接口并实现它的程序。

play: https://play.golang.org/p/F-T3S_wNNB

在上面创建了一个名为 `VowelsFinder` 的接口类型，它有一个方法 `FindVowels() []rune`。

然后创建一个类型 `MyString`。

我们将方法 `FindVowels() []rune` 添加到接收器类型 `MyString` 。现在 `MyString` 实现了接口 `VowelsFinder`。这和 Java 等其他语言不通，其中类必须明确声明使用 `implements` 关键字实现接口。但是在 Go 中不需要这样，在 Go 中是隐式实现的，如果类型包含接口中声明的所有方法，那么就表示实现了这个接口。

### 实际使用接口

我们将编写一个简单的程序，根据员工的个人工资计算公司的总支出。为简洁起见，我们假设所有费用均以美元计算（让我们也一起赚刀）。

假设我们公司有两种雇员，`Permanent` 和 `Contract`。

play：https://play.golang.org/p/5t6GgQ2TSU

### 接口内部表示

可以认为接口由元组内部表示（type, value）,`type` 是接口的基础具体类型，`value` 保存具体类型的值。

play：https://play.golang.org/p/Q40Omtewlh

### 空接口

具有零方法的接口称为空接口。它表示为 `interface{}`。由于空接口的方法为零，因此所有类型都实现了空接口。

play: https://play.golang.org/p/Fm5KescoJb

### 类型断言

类型断言是用于提取接口的基础值(underlying value)。

`i.(T)` 语法是用于获取 `i` 接口的具体类型的基础值。


```
s := i.(int)
```

play: https://play.golang.org/p/YstKXEeSBL

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

```
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

play：https://play.golang.org/p/o6aHzIz4wC

