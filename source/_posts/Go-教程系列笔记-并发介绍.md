---
title: '[Go 教程系列笔记] 并发介绍'
date: 2018-07-31 13:46:04
categories: Go
tags: 
    - Go
    - Go教程系列笔记
---

Go 是一种并发语言，而不是并行语言。在讨论如何在Go中处理并发之前，我们必须首先了解什么是并发以及它和并行性的区别。

### 什么是并发？

并发是一次处理大量事情的能力。举个例子：

> 让我们假设一个人慢跑。在他早晨慢跑时，他的鞋带解开了。现在这个人停止跑步，把鞋带帮好，然后又开始跑步。这就是并发的典型示例。这个人能够处理跑步和绑鞋带，也就是说，这个人能够同时处理很多事情。

<!--more -->

### 什么是并行性，它和并发性有什么不同？

**并行性是同时做了很多事情。**它可能听起来类似于并发，但它实际上是不同的。

让我们还是用同样的慢跑例子来更好地理解它。在这种情况下，我们假设这个人正在慢跑并且用iphone听音乐。在这种情况下，这个人正在慢跑并同时听音乐，也就是说，他正在做很多事情。这称为并行性。

### 并发与并行 - 一种技术观点

我们刚刚理解了什么是并发性以及它与真实世界示例的并行性有何不同。现在让我们从更技术的角度来看待它们，因为我们是极客嘛:)

假设我们正在编写一个 Web 浏览器。Web 浏览器具有各种组件。其中两个是网页显示和用于从互联网下载文件的下载程序。让我们假设我们已经构建好了我们的浏览器代码，使得每个组件都可以独立执行（这是使用Java等语言中的线程完成的，我们可以使用 Goroutine 实现这一点，稍后会详细介绍）。

当此浏览器在单核处理器中运行时，处理器将在浏览器的两个组件之间进行上下文切换。它可能正在下载文件一段时间，然后它可能会切换到呈现用户请求网页的HTML。这被称为并发。并发进程在不同的时间点开始，并且它们的执行周期重叠。在这种情况下，下载和渲染在不同的时间点开始，并且它们的执行重叠。

让我们假设同一个浏览器在多核处理器上运行。在这种情况下，文件下载组件和HTML呈现组件可以在不同的CPU中同时运行。这被称为并行。

![](http://blogcdn.weixinote.com/2019-08-14-15326597209062.jpg)

![](http://blogcdn.weixinote.com/2019-08-14-15326598785575.jpg)

并行性并不总是会导致更快的执行时间。这是因为并行运行的组件可能必须相互通信。例如，在我们的浏览器的情况下，当文件下载完成时，应该将其传达给用户，例如使用弹窗，此通信发送在负责下载组件和负责呈现用户界面的组件之间。并发系统中的通信开销很低。组件在多核中并行运行的情况下，这种通信开销很高。因此，并行程序并不总是会导致更快的执行时间。


### 在 Go 中支持并发性

并发是 Go 语言的固有部分。使用 goroutine 和 channel 在 Go 中处理并发。我们将在接下来的教程中详细讨论它们。
