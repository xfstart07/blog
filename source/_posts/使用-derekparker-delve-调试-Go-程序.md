---
title: 使用 derekparker/delve 调试 Go 程序
date: 2018-06-10 11:39:27
categories:
  - GO
tags:
  - GO
  - Debug
---

安装 delve 调试工具

1. 通过 `go get` 下载源码

```
go get -u github.com/derekparker/delve/cmd/dlv
```

2. 安装 dlv-cert 证书

在 `derekparker/delve/scripts` 下有 `gencert.sh` 脚本，执行它来生成证书。需要输入用户密码。
生成后在 Keychain 中可以看到。

3. 更新 xcode command-line

```
xcode-select --install
```

<!--more-->

但是升级后的CommandLineTools 的debug api有变化，目前暂时没法用兼容目前的delve。

4. 安装 dlv

解决上面 command-line 的问题。
在delve的目录：`$GOPATH/src/github.com/derekparker/delve` 中执行下面的命令：

```
git remote add aarzilli git@github.com:aarzilli/delve.git
git pull aarzilli debugserverfix
GO15VENDOREXPERIMENT=1 CERT=dlv-cert make install
```

执行完在 `$GOPATH/bin` 中就能看到 dlv 了。

### 使用 VSCode 调试
首先在 vscode 商店中搜索 Go 插件。

### 生成 launch.json 配置

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch",
      "type": "go",
      "request": "launch",
      "mode": "debug",
      "remotePath": "",
      "port": 2345,
      "host": "127.0.0.1",
      "program": "${fileDirname}",
      "env": {
        "GOPATH": "/Users/x/golang"   // 修改 GOPATH 路径
      },
      "args": [],
      "showLog": true
    }
  ]
}
```

配置完点击运行按钮就能直接运行调试了。

### 使用 Goland 调试
主要是需要将 Goland 中的 dlv 进行替换掉，换成 `$GOPATH/bin` 中的 dlv。

在 Goland 中运行调试会报错误，主要是应该 dlv 安装的原因。

```
could not launch process: EOF
```
