---
title: Docker 启动 RabbitMQ
date: 2018-07-05 10:08:37
categories: MQ
tags:
  - RabbitMQ
  - Docker
---

拉取 RabbitMQ

```
docker pull rabbitmq
```

启动 RabbitMQ：

```
docker run -d --hostname rabbit01 --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq:management
```

`-d` 以守护进程方式启动
`--hostname rabbit01` 设置 RabbitMQ 的主机名称。
`--name rabbit` 定义 RabbitMQ 容器名称
`-p 15672:15672` 将容器的端口映射到本机 15672 端口（RabbitMQ 的 web 端口）。
`-p 5672:5672` RabbitMQ 服务端口

最后指定了安装 RabbitMQ 的 Web管理插件版本：`rabbitmq:management`

打开浏览器输入：http://localhost:15672 可以看到 RabbitMQ 已经安装并启动起来了，使用默认的 `guest:guest` 用户名密码登录。

发送使用 `amqp://guest:guest@localhost:5672/`
