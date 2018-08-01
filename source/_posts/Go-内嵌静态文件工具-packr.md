---
title: "Go 内嵌静态文件工具 packr"
date: 2018-06-10 11:27:30
categories:
  - Go
tags:
  - Go
---


介绍一个简单实用的 Go 内嵌静态文件工具 packr。

### 安装

```
$ go get -u github.com/gobuffalo/packr/...
```

### 使用
使用 packr 打包静态文件非常简单，通过创建一个 box.

```
// templates 是相对路径，例子是在同一个目录下
box := packr.NewBox("./templates")

// 以字符串的形式获取静态文件
html := box.String("index.html")
fmt.Println("String获取:", html)

html, err := box.MustString("index.html")
if err != nil {
	log.Fatal(err)
}
fmt.Println("MustString获取", html)

// 以字节数组的形式获取静态文件
htmlByte := box.Bytes("index.html")
fmt.Println("Bytes: ", htmlByte)
// 对应的还有 MustBytes 方法
```

<!--more-->

packr 在查找文件时的解析规则：

* 在二进制文件中，在内存中查找文件
* 开发时，在本地查找文件

### 在 HTTP 中使用
因为 box 实现了 [`http.FileSystem`](https://golang.org/pkg/net/http/#FileSystem) 接口，所有可以直接用来提供静态文件访问

```
package main

import (
	"net/http"

	"github.com/gobuffalo/packr"
)

func main() {
	box := packr.NewBox("./templates")

	http.Handle("/", http.FileServer(box))
	http.ListenAndServe(":3000", nil)
}
```

使用 [`gorilla/mux`](https://github.com/gorilla/mux) 作为路由库，mux 也有提供静态文件访问的方式

```
r := mux.NewRouter()

box := packr.NewBox("./css")
r.PathPrefix("/css").Handler(http.StripPrefix("/css", http.FileServer(box)))
```

### 在渲染库(render)中使用
使用 `unrolled/render` 库来渲染资源，在初始化渲染选项中，将静态文件加入到 Asset 中。

```
var boxTemp = packr.NewBox("../templates")
var ren = render.New(render.Options{
	Directory: "templates",
	Asset: func(name string) ([]byte, error) {
	   // 返回指定路径名称的文件资源
		return boxTemp.Bytes("index.html"), nil
	},
	AssetNames: func() []string {
	   // 静态文件的路径名称
		return []string{"templates/index.html"}
	},
	Extensions:      []string{".html"},
})
```

使用

```
ren.HTML(w, http.StatusOK, "index.html", "")
```

### 打包命令
使用命令建立二进制文件

`packr build` 包括了 `go build`

`packr install` 包括了 `go install`

还可以使用 `go generate` 命令来生成静态资源文件(.go)，

```
package main

// 打包静态文件命令，生成 packr.go 文件
//go:generate packr

func main() {
	Run()
}
```

然后运行命令

```
go generate && go build
```

## 资源
https://github.com/gobuffalo/packr

https://github.com/unrolled/render

https://github.com/gorilla/mux