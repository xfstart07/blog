---
title: "[Go Websocket] Centrifugo 实时消息服务介绍"
date: 2018-06-10 10:52:39
categories:
  - Go
tags:
  - Go
  - Websocket
---

Centrifugo 是用 Go 实现的一套实时消息服务(Websockets 或 SockJS)，可以使用的场景很多，聊天，实时图表，通知，各种计算器，甚至游戏都可以。

Centrifugo 可以方便的集成到现有服务当中。

**简化方案示意图：**

{% img http://pa1so03xn.bkt.clouddn.com/images/0_hxTfRyG_JfKZaii8.png %}

<!--more-->

## 使用
下载地址：https://github.com/centrifugal/centrifugo/releases

查看 centrifugo 的帮助信息：

```
./centrifugo -h
```

centrifugo 的服务器节点需要具有密钥的配置文件，使用 `genconfig` 命令可以生成一个最小的配置文件:

```
./centrifugo genconfig
```

运行

```
 ./centrifugo --config=config.json // 默认运行端口 8000
```

这样就启动了一个实时消息服务了，就是那么简单。当然，想要了解更多还需要更深入的学习。

### 配置信息
几个重要的命令行选项:

* `--port` 用来绑定端口的（默认 8000）
* `--engine` 使用的引擎 `memory` 或 `redis`, 默认 `memory`
* `--admin` 启用管理 websocket 端点的 web 界面
* `--web` 提供 web 界面

ps. `--web` 启用 web 界面的同时需要在 config 文件中配置 `admin_password` 和 `admin_secret` 选项，这两个选项是 web 界面的登陆密码和授权令牌密钥。

运行命令类似:

```
./centrifugo --config=config.json --admin --web --port 8088
```

#### 配置文件

```
{
  "secret": "secret",
  "admin_password": "123456",
  "admin_secret": "123456",
  "publish": true,
  "watch": true,
  "presence": true,
  "join_leave": true,
  "history_size": 100,
  "history_lifetime": 3600,
  "recover": true,
  "history_drop_inactive": true,
  "namespaces": [
    {
      "name": "public",
      "publish": true,
      "watch": true,
      "presence": true,
      "join_leave": true,
      "history_size": 10,
      "history_lifetime": 30
    }
  ],
  "log_level": "debug"
}
```

* `secret` 密钥是唯一必要的选项。
* `connection_lifetime` 客户端连接到期的秒数。
* `watch` centrifugo 会另外发布消息到管理员通道（这些消息可以在 web message 标签中看到），默认是 `false`。需要注意，这个选项只推荐在开发环境时使用。
* `publish` 允许客户直接（从客户端）发布消息到频道。
* `anonymous` 启动匿名访问（连接参数中具有空的用户ID）
* `presence` 用户在当前通道订阅的结构信息
* `join_leave` 当用户在通道上订阅(取消订阅)时，发送消息。
* `history_size` 通道的历史保留记录数
* `history_lifetime` 通道的历史记录保留时长
* `recover` 启动时 centrifugo 将尝试恢复丢失的消息，根据 `history_size` 和 `history_lifetime`。
* `history_drop_inactive` 可在使用通道消息历史记录时大幅减少资源使用情况（引擎内存使用情况，消息传播）。
* `debug` 布尔值，开发模式

centrifugo 配置文件支持 `json`， `toml`，`yaml` 格式。

#### 高级配置
**client_channel_limit**

默认值 128

设置单个客户端可以拥有的最大数量的不同通道订阅

**max_channel_length**

默认值 255

设置通道名称的最大长度。

**默认端点**

websocket 的端点

```
ws://localhost:8000/connection/websocket
```

** API端点 **

```
http://localhost:8000/api/
```

## 通道
通道是消息的路由。

用户可以订阅通道，用来接收消息或发布消息。通道的名称长度默认是 255 个字符（可以配置）。

其次，`:`，`#`，`&`，`$` 字符在通道名称中有特殊作用。

## 服务接口
API接口 `http://localhost:8000/api/`，用于后端发布、广播、取消订阅等操作。
