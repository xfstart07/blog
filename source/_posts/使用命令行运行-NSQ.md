---
title: "[MQ] 使用命令行运行 NSQ"
date: 2018-08-09 15:32:19
categories:
  - MQ
tags:
  - MQ
  - NSQ
---

## 安装

**下载**

当前(20180809)版本: 1.0.0-compat

mac: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-1.0.0-compat.darwin-amd64.go1.8.tar.gz

linux: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-1.0.0-compat.linux-amd64.go1.8.tar.gz

**解压**

```bash
tar xvf nsq.tar.gz
```

<!-- more -->

## 运行

### nsqlookupd

```bash
bin/nsqlookupd -tcp-address=127.0.0.1:4160 -http-address=127.0.0.1:4161 -broadcast-address="127.0.0.1"
```

* `-broadcast-address` lookup 节点的地址（默认是 OS 名称, 例如 `mac.local`）
* `-tcp-address` TCP 客户端监听的地址，格式`<addr>:port`
* `-http-address` HTTP 客户端监听的地址，格式`<addr>:port`
* `-log-prefix` 日志前缀，默认 `"[nsqlookupd] "`

### nsqd

```bash
bin/nsqd -broadcast-address="127.0.0.1" -http-address="127.0.0.1:4151" -lookupd-tcp-address=127.0.0.1:4160 -tcp-address="127.0.0.1:4150"
```

* `-data-path` 存储消息的地址
* `-broadcast-address` 在 lookup 注册的地址（默认是 OS 名称, 例如 `mac.local`）
* `-http-address` HTTP 客户端监听的地址
* `-tcp-address` TCP 客户端监听的地址
* `-lookupd-tcp-address` lookup 的 TCP 地址（可以写多个）

### admin

```bash
bin/nsqadmin -lookupd-http-address=127.0.0.1:4161
```

* `-lookupd-tcp-address` lookup 的 TCP 地址（可以写多个）