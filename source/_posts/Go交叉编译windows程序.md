---
title: Go交叉编译windows程序
date: 2020-01-10 16:05:35
categories:
    - GO
tags:
    - 交叉编译
    - windows
---

在Mac下编译一个调用了 sqlite3 的程序。

### 安装 sqlite3

运行

```bash
go get github.com/mattn/go-sqlite3
go install github.com/mattn/go-sqlite3
```

<!--more-->

#### windows安装sqlite3

请访问 [SQLite](http://www.sqlite.org/download.html) 下载页面，从 Windows 区下载预编译的二进制文件。

您需要下载 sqlite-tools-*.zip 和 sqlite-dll-*.zip 压缩文件。然后创建文件夹 `C:\sqlite`，并在此文件夹下解压上面两个压缩文件，将得到 sqlite3.def、sqlite3.dll 和 sqlite3.exe 文件。

添加 `C:\sqlite` 到 PATH 环境变量，最后在命令提示符下，使用 sqlite3 命令，就能进入sqlite了。

### 交叉编译

在启用CGO_ENABLED的情况下，尝试使用下面命令进行Windows平台的交叉编译：

```bash
$ CGO_ENABLED=1 GOOS=windows GOARCH=386 go build -x -v -ldflags "-s -w"
```

出现错误如下：

```bash
# runtime/cgo
gcc_libinit_windows.c:7:10: fatal error: 'windows.h' file not found
```

#### 安装mingw-w64

```bash
brew install mingw-w64
```

#### 编译64位

```bash
CGO_ENABLED=1 CC=x86_64-w64-mingw32-gcc CXX=x86_64-w64-mingw32-g++ \ 
 GOOS=windows GOARCH=amd64 go build
```

#### 编译x86

```bash
$ CGO_ENABLED=1 CC=i686-w64-mingw32-gcc CXX=i686-w64-mingw32-g++ \ 
 GOOS=windows GOARCH=386 go build 
```

### windows好用的终端工具

ConEmu，比 cmd 好用N倍。

### Ref

* https://conemu.github.io/