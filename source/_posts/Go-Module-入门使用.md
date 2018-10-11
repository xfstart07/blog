---
title: Go Module 入门使用
date: 2018-10-10 20:38:30
categories: Go
tags:
  - Go
  - module
---

对于Go的版本管理主要用过 glide，下面介绍 Go 1.11 之后官方支持的版本管理工具 mod。关于 mod 官方给出了三个命令 `go help mod`、`go help modules`、`go help module-get` 帮助了解使用。

## 设置 GO111MODULE

可以用环境变量 `GO111MODULE` 开启或关闭模块支持，它有三个可选值：off、on、auto，默认值是 auto。

* `GO111MODULE=off` 无模块支持，go 会从 GOPATH 和 vendor 文件夹寻找包。
* `GO111MODULE=on` 模块支持，go 会忽略 GOPATH 和 vendor 文件夹，只根据 go.mod 下载依赖。
* `GO111MODULE=auto` 在 $GOPATH/src 外面且根目录有 go.mod 文件时，开启模块支持。

在使用模块的时候，GOPATH 是无意义的，不过它还是会把下载的依赖储存在 $GOPATH/pkg/mod 中，也会把 go install 的结果放在 $GOPATH/bin 中。

<!-- more -->

## Go Mod 命令

```
download    download modules to local cache (下载依赖的module到本地cache))
edit        edit go.mod from tools or scripts (编辑go.mod文件)
graph       print module requirement graph (打印模块依赖图))
init        initialize new module in current directory (再当前文件夹下初始化一个新的module, 创建go.mod文件))
tidy        add missing and remove unused modules (增加丢失的module，去掉未用的module)
vendor      make vendored copy of dependencies (将依赖复制到vendor下)
verify      verify dependencies have expected content (校验依赖)
why         explain why packages or modules are needed (解释为什么需要依赖)
```

## Go Mod 使用

### 创建 go.mod 文件

在一个新的项目中，需要执行`go mod init` 来初始化创建文件`go.mod`，`go.mod` 中会列出所有依赖包的路径和版本。

```
module github.com/xfstart07/watcher

require (
	github.com/apex/log v1.0.0
	github.com/fatih/color v1.7.0 // indirect
	github.com/fsnotify/fsnotify v1.4.7
	github.com/go-ini/ini v1.38.2
	github.com/go-kit/kit v0.7.0
	github.com/go-logfmt/logfmt v0.3.0 // indirect
```

`indirect` 表示这个库是间接引用进来的。

`go mod vendor` 命令可以在项目中创建 `vendor` 文件夹将依赖包拷贝过来。

`go mod download` 命令用于将依赖包缓存到本地Cache起来。

### 显示所有Import库信息

```
go list -m -json all
```

* `-json` JSON格式显示
* `all` 显示全部库

## Mod Cache 路径

默认在`$GOPATH/pkg` 下面：

```
$GOPATH/pkg/mod
```

我们来看看一个项目下载下来的文件形式：

```
➜  mod ls -lh cache/download/github.com/go-kit/kit/@v/
total 3016
-rw-r--r--  1 a1  staff     7B Sep 29 15:37 list
-rw-------  1 a1  staff    50B Sep 29 15:37 v0.7.0.info
-rw-------  1 a1  staff    29B Sep 29 15:37 v0.7.0.mod
-rw-r--r--  1 a1  staff   1.5M Sep 29 15:37 v0.7.0.zip
-rw-r--r--  1 a1  staff    47B Sep 29 15:37 v0.7.0.ziphash
```

可以看出项目库会对每个版本创建一个文件夹，文件夹下有对于版本的信息。

## 资料

* [初窥Go module](https://tonybai.com/2018/07/15/hello-go-module/)
* [Introduction to Go Modules](https://roberto.selbach.ca/intro-to-go-modules/)
* https://ieevee.com/tech/2018/08/28/go-modules.html
* https://colobu.com/2018/08/27/learn-go-module/
* https://studygolang.com/articles/13895