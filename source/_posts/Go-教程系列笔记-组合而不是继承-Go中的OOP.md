---
title: '[Go 教程系列笔记]组合而不是继承-Go中的OOP'
date: 2019-08-14 15:37:16
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

Go不支持继承，但它确实支持组合。组合的通用定义是“放在一起”。组合的一个例子是汽车。汽车由车轮，发动机和各种其他部件组成。

### 通过嵌入结构组成

Go中的组合可以通过将一种结构类型嵌入到另一种结构类型中来实现。

博客文章是一个完美的组合示例。每篇博文都有标题，内容和作者信息。这可以使用组合完美地表示。在本教程的后续步骤中，我们将了解如何完成此操作。

<!-- more -->

让我们首先创建`author`结构。

```go
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}
```

在上面的代码片段中，我们创建了一个author带字段的结构，firstName，lastName和bio。我们还添加了一个`fullName()`带有`author`接收器类型的方法，它返回作者的全名。

下一步是创建`post`结构。

```go
type post struct {  
    title     string
    content   string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.author.fullName())
    fmt.Println("Bio: ", p.author.bio)
}
```

`post`结构有字段 title，content。它还有一个嵌入式匿名字段`author`。该字段表示`post`结构由`author`组成。现在`post`结构可以访问`author`结构的所有字段和方法。我们还在`post`结构中添加了details()方法，用于打印作者的标题，内容，完整名称和生物。

每当一个struct字段嵌入另一个struct时，Go为我们提供了访问嵌入字段的选项，就好像它们是外部结构的一部分一样。这意味着`p.author.fullName()`在行中没有。上面的代码第 10 行中可以替换为`p.fullName()`。因此，该details()方法可以重写如下，

```go
func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}
```

现在我们已经准备好author和post结构，让我们通过创建一个博客文章来完成这个程序。

```go
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post1.details()
}
```

[Run in playground](https://play.golang.org/p/sskWaTpJgr)

上面程序中的主要功能是在第31行号中创建一个新作者。在第36行创建一个新文章。并嵌入式`author`。这个程序打印。

```
Title:  Inheritance in Go  
Content:  Go supports composition instead of inheritance  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast 
```

### 嵌入 slice 结构

我们可以把这个例子进一步修改，创建使用一个网站有一系列博客文章:)。

让我们先定义`website`结构。请在现有程序的主要功能上方添加以下代码并运行它。

```go
type website struct {  
        []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website\n")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}
```

在添加上面的代码后运行上面的程序时，编译器会抱怨以下错误，

```
main.go:31:9: syntax error: unexpected [, expecting field name or embedded type 
```

此错误指向嵌入的结构切片`[]post`。原因是不可能匿名嵌入切片。字段名称是必需的。所以让我们修复这个错误并使编译器满意。

```go
type website struct {  
        posts []post
}
```

我已将字段名称`posts`添加到 []post。

现在让我们修改`main.go`并为我们的新网站创建一些帖子。

修改主要功能后的完整程序如下所示，

```go
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

type website struct {  
 posts []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website\n")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post2 := post{
        "Struct instead of Classes in Go",
        "Go does not support classes but methods can be added to structs",
        author1,
    }
    post3 := post{
        "Concurrency",
        "Go is a concurrent language and not a parallel one",
        author1,
    }
    w := website{
        posts: []post{post1, post2, post3},
    }
    w.contents()
}
```

[Run in playground](https://play.golang.org/p/gKaa0RbeAE)

在上面的`main.go`中，我们创建了一个作者`author1`和三个帖子post1，post2并且post3。最后我们在第63行创建了网站 w。通过嵌入这3个帖子并在下一行显示内容。

该程序将输出，

```
Contents of Website

Title:  Inheritance in Go  
Content:  Go supports composition instead of inheritance  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast

Title:  Struct instead of Classes in Go  
Content:  Go does not support classes but methods can be added to structs  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast

Title:  Concurrency  
Content:  Go is a concurrent language and not a parallel one  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast  
```