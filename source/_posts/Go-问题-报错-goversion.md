---
title: '[Go 问题] 报错 "flag provided but not defined: -goversion"'
date: 2018-08-29 11:47:37
categories: Go
tags:
  - Go
  - Bug
---

今天 Jenkins 打包 go 项目的时候发生了一个错误，提示：“flag provided but not defined: -goversion”。

原因是因为系统中安装了两个 go 版本。

<!-- more -->

### 首先

通过命令查询 go 的目录：

```
# whereis go
go: /usr/bin/go /usr/local/go
```

在系统中存在两个go的运行目录：`/usr/bin/go` 和 `/usr/local/go`

### 解决

我们可以将其中一个版本删除掉。我的系统是 centos，`/usr/bin/go` 目录就是通过系统 `yum` 的方式安装的，可以使用 `yum remove go` 的方式卸载掉。

或者删除通过源码安装的 `/usr/local/go`。但是在删除之后还要记得修改 profile 中的配置。

因为我的配置是放在 `/etc/profile` 下：

```
export GOROOT=/usr/local/go
export GOPATH=/data/go
```

然后 `source /etc/profile` 生效一下配置。