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

### 使用 supervisor 管理 nsq 进程

首先添加 supervisor 的配置文件

#### nsqlookupd 的配置文件

```ini
[program:nsqlookupd]
command =nsqpath/bin/nsqlookupd run
autostart=true
startsecs=5
autorestart=true
startretries=3
user=nobody
redirect_stderr = true
stdout_logfile_maxbytes=100MB
stdout_logfile_backups=20
stdout_logfile=/var/log/nsq/lookup.log
stopasgroup=true
killasgroup=true
stopsignal=QUIT

[include]
files = /etc/supervisord.d/nsqlookupd.ini
```

**说明：**

* `autostart` 自动启动
* `startsecs` 启动5秒后没有异常退出，就表示进程正常启动了，默认为1秒
* `autorestart` 程序退出后自动重启
* `startretries` 启动失败自动重试次数，默认是3
* `user` 用哪个用户启动进程
* `redirect_stderr` 把stderr重定向到stdout，默认false
* `stdout_logfile_maxbytes` stdout 日志文件大小，默认50MB
* `stdout_logfile_backups` stdout 日志文件备份数，默认是10
* `stdout_logfile` stdout 日志路径
* `stopasgroup` 默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
* `killasgroup` 默认为false，向进程组发送kill信号，包括子进程
* `stopsignal` 进程停止信号，可以为TERM, HUP, INT, QUIT, KILL, USR1, or USR2等信号，默认为TERM

nsqd 和 admin 的配置基本一样就不在贴了。

#### supervisor 更新

在添加好配置之后，因为 supervisor 已经运行着，是不能停掉的，可以通过下面的命令更新：

```
supervisorctl update
```