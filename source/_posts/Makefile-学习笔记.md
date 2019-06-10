---
title: Makefile 学习笔记
date: 2019-06-10 15:23:54
categories: Tools
tags: makefile
---

`make`命令执行时，需要一个 `Makefile`文件，以告诉`make`命令需要怎么样的去编译和链接程序。

> Makefile 类似像带有上下文的 shell 命令记录文本。

### `Makefile`的规则

    target ... : prerequisites ...
        command
        ...

**target**

可以是一个object file（目标文件），也可以是一个执行文件，还可以是一个标签（Label）。

<!-- more -->

**prerequisites**

生成该`target`所依赖的文件或`target`

**command**

该`target`要执行的命令（任意的shell命令）

这是一个文件的依赖关系，也就是说，`target`这一个或多个的目标文件依赖于`prerequisites`中的文件， 其生成规则定义在`command`中。说白一点就是说:

> prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。

这就是makefile的规则，也就是makefile中最核心的内容。

### Makefile 中使用变量

[**变量文档**](https://seisman.github.io/how-to-write-makefile/variables.html)

在使用变量时在前面加上`$` 符号，最好使用小括号`()`包括起来。

- `=` 赋值
- `:=` 被赋值中有变量时，在这行语句中是什么值就是什么值。后面改变了这里的值也不会改变。
- `?=` 如果前面定义了，那么语句什么也不做。
- `+=` 用来给一个已经赋值的变量追加赋值。

定义：

    # Golang Flags
    GOPATH ?= $(shell go env GOPATH)
    GO=go

这里定义了两个变量，`GOPATH` 和 `GO` 。

调用：

    run-server: vet-server ## Runs server.   
    		@echo Running server for development   
    		$(GO) run $(SERVER_FILE)

### .PHONY 定义

    .PHONY: run-server

`.PHONY` 表示 run-server 是个伪目标文件。相当于预先定义这些名称是目标文件。

### 编写 Makefile 注意事项

- makefile 文件中的缩进是 tab，而不能用空格替代。否则会报`*** missing separator. Stop.` 错误。

## Ref

- [https://seisman.github.io/how-to-write-makefile/overview.html](https://seisman.github.io/how-to-write-makefile/overview.html)
- [http://www.ruanyifeng.com/blog/2015/02/make.html](http://www.ruanyifeng.com/blog/2015/02/make.html)