---
title: golang 数据结构
date: 2019-08-13 09:16:44
categories: Go
tags: 
    - Go
    - 数据结构
---

golang 常用数据结构

* struct
* array
* slice
* map

### struct

基本用法

<!--more-->

```go
// 定义结构体
type Person struct {
    Name string
    Age int
}

// 结构体值接收者方法
func (p Person) getName() string {
    return p.Name 
}

// 结构体指针接收者方法
func (p *Person) setName(name string) string {
    p.Name = name 
}

// 声明结构体
var person Person 
var person *Person 
person := Person{Name: 'leon'}
person := &Person{Name: 'leon'}
```

结构体还可以嵌套，只声明类型，而不指定名字，则默认包涵该类型所有字段。也可以指定名字。

```go

type Human struct {
    Person
}

var h Human
h.Name // 可以直接调用
h.getName()

type Human struct {
    male Person
}

var h Human
h.male.Name // 需要通过名字调用
h.male.getName()
```

### array

数组的长度不可改变，声明时需要指定数组长度和类型。

```go
var arr [20]int
arr := [2]int{1,2}
arr := [...]int{1,3} // 根据写入的个数自动计算长度

arr[1] = 1

// 转换为 slice
sarr := arr[0:2]
```

### slice

* [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)
* [Go Slices: usage and internals 翻译](https://blog.go-zh.org/go-slices-usage-and-internals)

slice 是可以动态增长的数组。声明时可以指定长度(len)和容量(cap)。当 cap 一般是 *2 增长。

slice 可以用内建函数 make 来创建，具体可填参数：make(Type, len, cap)。

```go
var arr []int // arr == nil 
arr := []int{1,2}
arr := make([]int, 10) // 指定 len=10，默认 cap=10
arr := make([]int, 10, 20) // 指定 len=10，cap=20

arr = arr[2:4] // 取子slice，取 2，3
arr = arr[:3] // 取 0,1,2
arr = arr[3:] // 取 3及后面所有元素 3,....

arr = arr[:0] // 复用 slice, len, cap = 0

arr = append(arr, 1) // 通过 append 给 slice 追加元素

for index, value := range arr {
    fmt.Printf("2**%d = %d\n", index, value)
}
```

说一下两种声明的区别：

```go
var arr []int // 声明了类型，但并没有初始化零值，在 json 编码时是 null
arr := []int // 声明了类型并初始化，在 json 编码时是 []
```

#### slice 在源码中具体的数据结构是什么样？

从数据结构上来看，slice 本质是一个struct，包含三部分：头指针、长度、容量。任何对 slice 长度的改变，都会返回一个新 slice（因为内容的长度发生了变化）。其实头指针指向一个数组，如果你使用 append 函数来追加元素时，如果数组使用完了，append 函数会自动的创建一个更长的新数组，然后返回一个新的 struct，包含新数组的长度和容量。

slice 结构体在源码 `runtime/slice.go` 中的定义，array 是指向数组的指针，len、cap 是长度和容量。
```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

### map

基本使用

```go
var hash map[string]string
hash := make(map[string]string)

// 赋值
hash["a"] = "a"

// 删除
delete(hash, "a")

// 赋值并返回一个是否存在标识，ok 是一个 bool
v, ok := hash["a"]
```

# Ref

* [Laisky Golang 学习笔记](https://blog.laisky.com/p/golang/#lplZOstuwJfvhat)
* [Go Data Structures](https://research.swtch.com/godata)
* [Go Data Structures Interface 翻译](https://github.com/studygolang/GCTT/blob/master/published/tech/20180326-Go-Data-Structures-Interface.md)