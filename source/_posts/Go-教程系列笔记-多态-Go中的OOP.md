---
title: '[Go 教程系列笔记]多态-Go中的OOP'
date: 2019-08-14 15:39:01
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---


Go 中的多态性是在接口的帮助下实现的。正如我们已经讨论过的，接口可以在Go 中隐式实现。如果类型定义了接口中声明的所有方法，则类型实现接口。让我们看看在接口的帮助下如何在Go中实现多态性。

### 使用接口的多态性

任何定义了接口所有方法的类型都被称为隐式实现该接口。

接口类型的变量可以包含实现接口的任何值。接口的这个属性用于在Go中实现多态。

让我们在计算组织净收入的程序的帮助下理解Go中的多态性。为简单起见，我们假设这个想象中的组织有两种项目的收入。固定账单，时间和材料。该组织的净收入按这些项目的收入总和计算。为了简化本教程，我们假设货币是美元，我们不会处理美分。它将使用int。（我建议阅读 https://forum.golangbridge.org/t/what-is-the-proper-golang-equivalent-to-decimal-when-dealing-with-money/413 以了解如何处理美分。）

<!-- more -->

我们首先定义一个接口Income。

```go
type Income interface {  
    calculate() int
    source() string
}
```

Income上面定义的接口包含两个方法 `calculate()` 计算并返回来自源的收入，和`source()` 返回源的名称。

接下来让我们为`FixedBilling`项目类型定义一个结构。

```go
type FixedBilling struct {  
    projectName string
    biddedAmount int
}
```

该`FixedBilling`项目有两个字段 projectName，表示项目的名称，以及`biddedAmount`组织为项目出价的金额。

该`TimeAndMaterial`结构将代表时间和材料类型的项目。

```go
type TimeAndMaterial struct {  
    projectName string
    noOfHours  int
    hourlyRate int
}
```

该`TimeAndMaterial`结构有三个字段名称`projectName`，  `noOfHours`和`hourlyRate`。

下一步是定义这些结构类型的方法，这些方法计算并返回实际收入和收入来源。

```go
func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}
```

就`FixedBilling`项目而言，收入只是项目的投标金额。因此我们从`FixedBilling`类型的`calculate()`方法返回它。

在`TimeAndMaterial`项目的例子里，产品的收入 `noOfHours`和`hourlyRate`，从具有`TimeAndMaterial` 接收器类型的方法`calculate()` 返回此值。

我们将该项目的名称作为该`source()`方法的收入来源返回。

由于两者`FixedBilling`和`TimeAndMaterial`结构都提供了接口calculate()和source()方法的定义Income，因此两个结构都实现了`Income`接口。

让我们声明`calculateNetIncome`将计算和打印总收入的函数。

```go
 func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}
```

这 `calculateNetIncome` 函数接受 `[]Income` 作为参数的接口。它通过迭代切片并在每个项目上调用方法 `calculate()` 来计算总收入。它还通过调用`source()`方法显示收入来源。根据具体类型的`Income`接口，不同`calculate()`和`source()`方法将被调用。因此我们在`calculateNetIncome`函数中实现了多态性。

