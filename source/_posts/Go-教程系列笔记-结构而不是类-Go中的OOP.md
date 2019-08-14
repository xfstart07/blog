---
title: '[Go 教程系列笔记]结构而不是类-Go中的OOP'
date: 2019-08-14 15:34:31
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

### Go 面向对象？

Go 不是纯粹的面向对象编程语言。摘自Go的常见问题解答，回答了Go是否面向对象的问题。

```
Yes and no. Although Go has types and methods and allows an object-oriented style of programming, there is no type hierarchy. The concept of “interface” in Go provides a different approach that we believe is easy to use and in some ways more general. There are also ways to embed types in other types to provide something analogous—but not identical—to subclassing. Moreover, methods in Go are more general than in C++ or Java: they can be defined for any sort of data, even built-in types such as plain, “unboxed” integers. They are not restricted to structs (classes).  
```

在接下来的教程中，我们将讨论如何使用 Go实现面向对象的编程概念。与其他面向对象的语言（如Java）相比，它们中的一些在实现上有很大不同。

<!-- more -->

### 结构而不是类

Go不提供类，但它确实提供了结构。可以在结构上添加方法。这提供了将数据和方法捆绑在一起的行为，类似于类。

让我们马上开始一个例子，以便更好地理解。

我们将在此示例中创建一个自定义包，因为它有助于更​​好地理解结构如何成为类的有效替代。

在 Go工作区内创建一个文件夹并命名 `oop`。在 `oop` 里面创建一个子文件夹 `employee`。在 `employee` 文件夹中，创建一个名为 employee.go. 的文件，文件夹结构看起来像，

```
workspacepath -> oop -> employee -> employee.go
```

employee.go 内容：

```go
package employee

import (  
    "fmt"
)

type Employee struct {  
    FirstName   string
    LastName    string
    TotalLeaves int
    LeavesTaken int
}

func (e Employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.FirstName, e.LastName, (e.TotalLeaves - e.LeavesTaken))
}
```

在上面的程序中，第一行指定此文件属于 employee包。Employee结构在第 7行号中声明。在第 14 行将一个名为的方法`LeavesRemaining`添加到Employee的结构中。这将计算并显示员工剩余的离职数量。现在我们有一个结构和一个方法，它运行在一个类似于类的结构上。

在`oop`文件夹中创建一个名为`main.go`的文件。现在文件夹结构看起来像，

```
workspacepath -> oop -> employee -> employee.go  
workspacepath -> oop -> main.go  
```

main.go 内容如下：

```go
package main

import "oop/employee"

func main() {  
    e := employee.Employee {
        FirstName: "Sam",
        LastName: "Adolf",
        TotalLeaves: 30,
        LeavesTaken: 20,
    }
    e.LeavesRemaining()
}
```

我们在第 3 行引入`employee`包。在`main.go`的第 12 行结构体` Employee` 调用方法 `LeavesRemaining()`。

此程序无法在 playground 上运行，因为它有自定义程序包。如果你在本地运行这个程序，可以在 `workspacepath/bin/oop` 文件夹下运行 `go install oop`。这个命令将打印输出：

```
Sam Adolf has 10 leaves remaining 
```

### New() 函数替代构造函数

我们上面写的程序看起来不错，但它有一个小问题。让我们看看当我们定义零值的Employee时会发生什么。将内容更改为`main.go`以下代码，

```go
package main

import "oop/employee"

func main() {  
    var e employee.Employee
    e.LeavesRemaining()
}
```

我们所做的唯一改变是Employee在第6行创建一个零值。该程序将输出，

```
has 0 leaves remaining
```

如您所见，使用零值创建的Employee变量不可用。它没有有效的名字，姓氏，也没有有效的休假详情。

在像Java这样的其他OOP语言中，这个问题可以通过使用构造函数来解决。可以使用参数化构造函数创建有效对象。

Go不支持构造函数。如果一个类型的零值不可用，那么程序员应该去掉导出，以防止其他包访问，并且还提供一个方法叫NewT(parameters)，其初始化类型T与所需的值。Go中的一个约定是命名一个函数 NewT(parameters)，它创建一个类型T。这将像一个构造函数。如果包只定义了一个类型，那么Go中的一个约定就是命名这个函数New(parameters)而不是NewT(parameters)。

让我们对我们编写的程序进行更改，以便每次创建员工时都可以使用。

第一步是取消导出Employee结构并创建一个New()函数来创建Employee。

用 employee.go 以下内容替换代码，

```go
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

我们在这里做了一些重要的改变。我们已将Employee struct 的起始字母改为小写 e，即我们已将 `type Employee struct`更改为`type employee struct`。通过这样做，我们已成功取消导出 employee 结构并阻止从其他包访问。除非特定需要导出它们，否则将未导出结构的所有字段都取消导出是一种很好的做法。由于我们不需要employee包外的任何结构的字段，因此我们也取消了所有字段的输出。

我们已经更改了在`LeavesRemaining()`方法中相应地字段名称。

现在，由于employee未导出，因此无法从其他包创建Employee类型的值。因此我们在第14行 New() 提供了一个导出函数。将所需参数作为输入并返回新创建的员工。

该程序仍然需要进行更改以使其工作，但是让我们运行它来了解到目前为止更改的效果。如果运行此程序，它将失败并出现以下编译错误，

```
go/src/constructor/main.go:6: undefined: employee.Employee  
```

这是因为我们已经取消导出Employee，因此编译器会抛出此类型未定义的错误。完美。正是我们想要的。现在没有其他包能够创建employee零值。我们已成功阻止创建不可用的员工结构值。现在创建员工的唯一方法是使用该New功能。

用以下内容替换内容main.go，

```go
package main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

对此文件的唯一更改是第 6行。我们通过将所需参数传递给New函数来创建新员工。

以下是进行所需更改后的两个文件的内容，

employee.go 内容

```go
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

main.go 内容

```go
package main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

运行此程序将输出，

```
Sam Adolf has 10 leaves remaining  
```

因此，您可以理解尽管Go不支持类，但可以有效地使用结构代替类，并且使用签名方法 `New(parameters)`替换构造函数的位置。