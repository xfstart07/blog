---
title: AMQP 0-9-1 模型说明
date: 2018-07-04 17:58:38
categories: MQ
tags:
  - MQ
  - RabbitMQ
---

# # AMQP 0-9-1 Model Explained AMQP 0-9-1 模型说明
[RabbitMQ - AMQP 0-9-1 Model Explained](http://www.rabbitmq.com/tutorials/amqp-concepts.html)

### About This Guide
### 关于这个指南

This guide provides an overview of the the AMQP 0-9-1 protocol, one of the protocols supported by RabbitMQ.
这个指南提供了概述 AMQP 0-9-1 协议。RabbitMQ 支持的协议之一。

<!--more-->

### What is AMQP 0-9-1?
### AMQP 0-9-1 是什么？

AMQP 0-9-1 (Advanced Message Queuing Protocol) is a messaging protocol that enables conforming client applications to communicate with conforming messaging middleware brokers.
AMQP 0-9-1 (高级消息队列协议)是一个消息协议，它符合客户端应用和消息中间代理进行通信。

### Brokers and Their Role
### 代理和它的角色

Messaging brokers receive messages from *publishers* (applications that publish them, also known as producers) and route them to *consumers* (applications that process them).
消息代理从*发布者*接收消息（发布它们的应用程序，也称为生成者），并且它们路由给*消费者*（处理它们的应用程序）。

Since it is a network protocol, the publishers, consumers and the broker can all reside on different machines.
由于它是一个网络协议，发布者，消费者和代理可以全部在不同的机器上。

### AMQP 0-9-1 Model in Brief
### AMQP 0-9-1 模型简介

The AMQP 0-9-1 Model has the following view of the world: messages are published to *exchanges*, which are often compared to post offices or mailboxes. Exchanges then distribute message copies to *queues* using rules called *bindings*. Then AMQP brokers either deliver messages to consumers subscribed to queues, or consumers fetch/pull messages from queues on demand.

AMQP 0-9-1 模型有以下观点：发布消息到交换机，这个通常和邮局或邮箱比较。交换机使用称为*绑定*规则复制消息分发到队列。那些 AMQP 代理将消息传递给订阅了队列的消费者。或者消费者根据需要从队列中获取/拉取消息。

![hello-world-example-routing](http://pa1so03xn.bkt.clouddn.com/hello-world-example-routing.png)


When publishing a message, publishers may specify various *message attributes* (message meta-data). Some of this meta-data may be used by the broker, however, the rest of it is completely opaque to the broker and is only used by applications that receive the message.
当发布一个消息，发布者可能指定各种消息属性（消息元数据）。代理可以使用部分的元数据。剩下的则完全对代理不透明，只能由接收消息的应用程序使用。

Networks are unreliable and applications may fail to process messages therefore the AMQP model has a notion of *message acknowledgements*: when a message is delivered to a consumer the consumer notifies the broker, either automatically or as soon as the application developer chooses to do so. When message acknowledgements are in use, a broker will only completely remove a message from a queue when it receives a notification for that message (or group of messages).

由于网络的不可靠，应用程序处理消息可能失败，因此 AMQP 模型有一个概念 *消息确认* ，当一个消息传递给消费者，消费者会自动回复或应用开发者通知代理。当消息确认正在使用，代理只有在收到该消息（或消息组）的通知才将消息完全从队列删除。

In certain situations, for example, when a message cannot be routed, messages may be *returned* to publishers, dropped, or, if the broker implements an extension, placed into a so-called "dead letter queue". Publishers choose how to handle situations like this by publishing messages using certain parameters.

在一些情况，例如，当一个消息不能路由时，消息可能返回给发布者，丢失，或者如果代理实现了扩展，放入一个叫做“死亡信件队列”。发布者可以通过特定参数发布消息来选择如何去处理解决这类情况。

Queues, exchanges and bindings are collectively referred to as *AMQP entities*.
队列，交换机和绑定统称为 AMQP 实体。

### AMQP is a Programmable Protocol
### AMQP 是一个可编程协议

AMQP 0-9-1 is a programmable protocol in the sense that AMQP 0-9-1 entities and routing schemes are primarily defined by applications themselves, not a broker administrator. Accordingly, provision is made for protocol operations that declare queues and exchanges, define bindings between them, subscribe to queues and so on.

AMQP 0-9-1 是一个可编程协议，在这个意义上AMQP 0-9-1 实体和路由方案主要由应用程序本身定义，而不是代理管理员。因此，规定了协议操作声明队列和交换机，定义它们之间的绑定，订阅队列等等。

This gives application developers a lot of freedom but also requires them to be aware of potential definition conflicts. In practice, definition conflicts are rare and often indicate a misconfiguration.

这个给应用开发者很大自由但是也要求它们知道潜在定义的约束。实际上，很少见定义约束，通常表明了配置错误。

Applications declare the AMQP 0-9-1 entities that they need, define necessary routing schemes and may choose to delete AMQP 0-9-1 entities when they are no longer used.

应用程序它们需要声明 AMQP 0-9-1 实体，定义必须的路由方案，当它们长时间不使用的情况也可以选择删除 AMQP 0-9-1 实体。

### Exchanges and Exchange Types
### 交换机和交换类型

*Exchanges* are AMQP entities where messages are sent. Exchanges take a message and route it into zero or more queues. The routing algorithm used depends on the exchange type and rules called *bindings*. AMQP 0-9-1 brokers provide four exchange types:
当发送消息时交换机是 AMQP 实体，交换机接收一条消息并将其路由到零个或多个队列中。这个路由算法依赖于交换类型并绑定路由。AMQP 0-9-1 代理提供4中交换类型：


| Name             | Default pre-declared names              |
| ---------------- | --------------------------------------- |
| Direct exchange  | (Empty string) and amq.direct           |
| Fanout exchange  | amq.fanout                              |
| Topic exchange   | amq.topic                               |
| Headers exchange | amq.match (and amq.headers in RabbitMQ) |

| 名称     | 默认预先声明的名称                      |
| -------- | --------------------------------------- |
| 直连交换 | (空字符串)或 amq.direct                 |
| 扇形交换 | amq.fanout                              |
| 主题交换 | amq.topic                               |
| 头部交换 | amq.match (and amq.headers in RabbitMQ) |


Besides the exchange type, exchanges are declared with a number of attributes, the most important of which are:

* Name
* Durability (exchanges survive broker restart)
* Auto-delete (exchange is deleted when last queue is unbound from it)
* Arguments (optional, used by plugins and broker-specific features)

除了交换类型，交换还有许多属性声明，其中最重要的是：

* 名称
* 持久化（代理重启，交换机还在）
* 自动删除（当最后一个队列被解除绑定是，交换机将被删除）
* 参数（可选，通过插件使用代理特定的功能）

Exchanges can be durable or transient. Durable exchanges survive broker restart whereas transient exchanges do not (they have to be redeclared when broker comes back online). Not all scenarios and use cases require exchanges to be durable.

交换机可以持久或短暂，代理重启后持久则交换机还在，短暂则不会（当代理还在线时，它们会重新声明）。不是所有的场景和情况都要求交换机必须持久。

### Default Exchange
### 默认交换机

The default exchange is a direct exchange with no name (empty string) pre-declared by the broker. It has one special property that makes it very useful for simple applications: every queue that is created is automatically bound to it with a routing key which is the same as the queue name.

默认交换机是代理预先声明的没有名字（空字符串）的直接交换机。它有一个特殊的属性，对于简单的应用程序非常有用，创建的每个队列都使用和队列相同的路由键自动绑定到它。

For example, when you declare a queue with the name of "search-indexing-online", the AMQP 0-9-1 broker will bind it to the default exchange using "search-indexing-online" as the routing key. Therefore, a message published to the default exchange with the routing key "search-indexing-online" will be routed to the queue "search-indexing-online". In other words, the default exchange makes it seem like it is possible to deliver messages directly to queues, even though that is not technically what is happening.

例如，当你声明一个队列的名字“search-indexing-online”，在 AMQP 0-9-1 代理使用 “search-indexing-online”作为路由键将其绑定到默认交换机上。因此，发送给具有路由键“search-indexing-online”的默认交换机的消息将被路由到队列“search-indexing-online”。换句话说，默认交换机看上去可能是直接传递消息到队列，即使技术上并非如此。

### Direct Exchange
### 直接交换机

A direct exchange delivers messages to queues based on the message routing key. A direct exchange is ideal for the unicast routing of messages (although they can be used for multicast routing as well). Here is how it works:

* A queue binds to the exchange with a routing key K
* When a new message with routing key R arrives at the direct exchange, the exchange routes it to the queue if K = R

直接交换机基于消息路由键传递消息给队列。直接交换机是理想单播路由消息（虽然它们也使用多播路由）。它是如何工作：

* 一个队列在路由键 K 去绑定交换机
* 当具有路由键 R 的消息到达直接交换机时，如果 K = R，交换机将路由它到对应队列。

Direct exchanges are often used to distribute tasks between multiple workers (instances of the same application) in a round robin manner. When doing so, it is important to understand that, in AMQP 0-9-1, messages are load balanced between consumers and not between queues.

直接交换机通常使用循环方式在多个工作人员（相同应用的实例）之间分配任务。当这样做时，重要去理解，在AMQP 0-9-1中,消息在多个消费者之间而不是在多个队列之间做负载均衡。

A direct exchange can be represented graphically as follows:
直接交换机可以用图形表示如下：

![exchange-direct](http://pa1so03xn.bkt.clouddn.com/exchange-declare.png)


### Fanout Exchange
### 扇形交换机

A fanout exchange routes messages to all of the queues that are bound to it and the routing key is ignored. If N queues are bound to a fanout exchange, when a new message is published to that exchange a copy of the message is delivered to all N queues. Fanout exchanges are ideal for the broadcast routing of messages.

扇形交换机将消息路由并绑定到所有的队列，并忽略路由键。如果N个队列绑定到扇形交换机，当新消息发布到交换机，交换机将拷贝消息传递到N个队列。扇形交换机是广播路由消息的理想选项。

Because a fanout exchange delivers a copy of a message to every queue bound to it, its use cases are quite similar:

* Massively multi-player online (MMO) games can use it for leaderboard updates or other global events
* Sport news sites can use fanout exchanges for distributing score updates to mobile clients in near real-time
* Distributed systems can broadcast various state and configuration updates
* Group chats can distribute messages between participants using a fanout exchange (although AMQP does not have a built-in concept of presence, so XMPP may be a better choice)

因为扇形交换机传递消息副本到每个绑定的队列，它使用的用例非常相似：

* 大型多人在线（MMO）游戏能使用它为排行榜更新或其他全局事件
* 体育消息网站能使用扇形交换机为移动客户端实时分发分数更新
* 分布式系统能广播各种状态和配置更新
* 群聊可以使用扇形交换机在参与者之间分发消息（虽然 AMQP 没有存在内置概念，因此 XMPP 可能是更好的选择）

A fanout exchange can be represented graphically as follows:
扇形交换机能用如下图形表示：

![exchange-fanout](http://pa1so03xn.bkt.clouddn.com/exchange-fanout.png)


### Topic Exchange
### 主题交换机

Topic exchanges route messages to one or many queues based on matching between a message routing key and the pattern that was used to bind a queue to an exchange. The topic exchange type is often used to implement various `publish/subscribe` pattern variations. Topic exchanges are commonly used for the multicast routing of messages.

主题交换机通过对消息的路由键和队列到交换机的绑定模式之间的匹配，将消息路由到一个或多个队列。主题交换机类型通常用于实现各种改变的`发布/订阅`模式。主题交换机通常使用消息的多播路由(multicast routing)。

Topic exchanges have a very broad set of use cases. Whenever a problem involves multiple consumers/applications that selectively choose which type of messages they want to receive, the use of topic exchanges should be considered.

主题交换机有非常广泛的用例，每当一个问题涉及多个消费者/应用程序，它们有选择地选择要接收的消息类型时，应该考虑使用主题交换机。

Example uses:

* Distributing data relevant to specific geographic location, for example, points of sale
* Background task processing done by multiple workers, each capable of handling specific set of tasks
* Stocks price updates (and updates on other kinds of financial data)
* News updates that involve categorization or tagging (for example, only for a particular sport or team)
* Orchestration of services of different kinds in the cloud
* Distributed architecture/OS-specific software builds or packaging where each builder can handle only one architecture or OS

示例用途：

* 分发与特定地理位置相关，例如，销售点
* 通过多个工作人员在后台任务处理完任务，每个工人处理特定一组任务。
* 股票价格更新（或更新其他类型的财务数据）
* 涉及类型或标签的新闻更新（例如，仅针对特定运动或团队）
* 在云中各种服务的协调
* 分布式架构/特定系统的软件构建或打包，每个构建器只能处理一个结构或系统。

### Headers Exchange
### 头部交换机

A headers exchange is designed for routing on multiple attributes that are more easily expressed as message headers than a routing key. Headers exchanges ignore the routing key attribute. Instead, the attributes used for routing are taken from the headers attribute. A message is considered matching if the value of the header equals the value specified upon binding.

头部交换机是为多个属性进行路由而设计的，这些属性比路由键更容易表达消息头部，头部交换机忽略路由键属性，相反，用于路由的属性取决于头部属性。如果头部的值等于特定绑定的值，则被认为消息是匹配的。

It is possible to bind a queue to a headers exchange using more than one header for matching. In this case, the broker needs one more piece of information from the application developer, namely, should it consider messages with any of the headers matching, or all of them? This is what the "x-match" binding argument is for. When the "x-match" argument is set to "any", just one matching header value is sufficient. Alternatively, setting "x-match" to "all" mandates that all the values must match.

可以使用多个头来将队列绑定到头部交换机来进行匹配，在这种情况下，代理需要来自应用程序的一些信息，换句话说，它应该考虑任何头部匹配的消息或全部消息？这就是”x-match”绑定参数的用途。当“x-match”参数设置为“any”时，只有一个匹配的标题值就足够了。或者，将“x-match”设置为“all”，强制所有值必须匹配。

Headers exchanges can be looked upon as "direct exchanges on steroids". Because they route based on header values, they can be used as direct exchanges where the routing key does not have to be a string; it could be an integer or a hash (dictionary) for example.

头部交换机能看成“direct exchanges on steroids（类固醇直接交换）”。因为他们基于头部值路由，它们能使用直接交换机，其中路由键比一定是字符串，例如，它也可以是整型或哈希（字典）。

### Queues
### 队列

Queues in the AMQP 0-9-1 model are very similar to queues in other message- and task-queueing systems: they store messages that are consumed by applications. Queues share some properties with exchanges, but also have some additional properties:

* Name
* Durable (the queue will survive a broker restart)
* Exclusive (used by only one connection and the queue will be deleted when that connection closes)
* Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes)
* Arguments (optional; used by plugins and broker-specific features such as message TTL, queue length limit, etc)

AMQP 0-9-1 模型中的队列和其他消息队列或任务队列系统非常相似：它们存储应用程序使用的消息。队列与交换机共享一些属性，但也有一些附加属性：

* 名称
* 持久化（队列在代理重启后还存在）
* 独家（只有一个连接使用，当连接关闭时将删除队列）
* 自动删除（当最后一个消费者取消订阅时，队列在最后一个消费者后删除）
* 参数（可选，通过插件或代理特定功能使用，例如：消息TTL（存活时间），队列长度限制等）

Before a queue can be used it has to be declared. Declaring a queue will cause it to be created if it does not already exist. The declaration will have no effect if the queue does already exist and its attributes are the same as those in the declaration. When the existing queue attributes are not the same as those in the declaration a channel-level exception with code 406 (PRECONDITION_FAILED) will be raised.

在使用队列之前需要先声明它。如果队列不存在，声明队列将导致它的创建。如果队列存在并且属性和声明中的属性相同，则声明将不起作用。当现有的队列属性与声明中的不同，将引发代码406（PRECONDITION_FAILED（先决条件失败））的通道级异常。

### Queue Names
### 队列名称

Applications may pick queue names or ask the broker to generate a name for them. Queue names may be up to 255 bytes of UTF-8 characters. An AMQP 0-9-1 broker can generate a unique queue name on behalf of an app. To use this feature, pass an empty string as the queue name argument. The generated name will be returned to the client with queue declaration response.

应用程序可以选择队列名称或要求代理为其生成名称。队列名称最多支持255个字节的UTF-8字符。AMQP 0-9-1 代理能代表应用生成唯一的队列名称。要使用这个功能，可以通过传一个空字符串队列名称参数。生成的名称将通过队列声明响应返回给客户端。

Queue names starting with "amq." are reserved for internal use by the broker. Attempts to declare a queue with a name that violates this rule will result in a channel-level exception with reply code 403 (ACCESS_REFUSED).

队列名称使用 “amq.” 开头是保留给代理内部使用的。尝试声明的队列名字违反了这条规则的结果将回复代码 403 的通道级的异常。

### Queue Durability
### 队列持久化

Durable queues are persisted to disk and thus survive broker restarts. Queues that are not durable are called transient. Not all scenarios and use cases mandate queues to be durable.

持久队列是持久化到磁盘，从而代理重启还存在。队列不持久称为瞬间。不是所有的场景和用例都要求队列持久化。

Durability of a queue does not make messages that are routed to that queue durable. If broker is taken down and then brought back up, durable queue will be re-declared during broker startup, however, only persistent messages will be recovered.

队列的持久不会使路由到该队列的消息持久。如果代理下线，然后重启启动，持久队列将在代理启动期间重新声明，但是只有持久化的消息才会被恢复。

### Bindings
### 绑定

Bindings are rules that exchanges use (among other things) to route messages to queues. To instruct an exchange E to route messages to a queue Q, Q has to be bound to E. Bindings may have an optional routing key attribute used by some exchange types. The purpose of the routing key is to select certain messages published to an exchange to be routed to the bound queue. In other words, the routing key acts like a filter.

绑定是交换机使用（除其他外）将消息路由到队列的规则。为了指示交换机 E 将消息路由到队列 Q，Q 必须绑定在 E。绑定在一些交换类型使用选项路由键属性。路由键的目的是选择某些发布到交换机的消息，将其路由到绑定的队列。换句话说，路由键像一个过滤器。

To draw an analogy:

* Queue is like your destination in New York city
* Exchange is like JFK airport
* Bindings are routes from JFK to your destination. There can be zero or many ways to reach it

举一个比喻：

* 队列像你在纽约的目的地
* 交换机像 JFK 飞机场
* 绑定是JFK到你目的地的路线(routes)，可以有零或许多方法到达它

Having this layer of indirection enables routing scenarios that are impossible or very hard to implement using publishing directly to queues and also eliminates certain amount of duplicated work application developers have to do.

有这种间接层使得路由方案是不可能或很难实现直接发布到队列，并且消除了应用开发者必须做的一些重复工作。

If AMQP message cannot be routed to any queue (for example, because there are no bindings for the exchange it was published to) it is either dropped or returned to the publisher, depending on message attributes the publisher has set.

如果 AMQP 消息不能路由到任何队列（例如，因为没有发布到它绑定的交换机），它将被删除或返回给发布者，这取决于发布者设置的消息属性。

### Consumers
### 消费者

Storing messages in queues is useless unless applications can consume them. In the AMQP 0-9-1 Model, there are two ways for applications to do this:

* Have messages delivered to them ("push API")
* Fetch messages as needed ("pull API")

将消息存储在队列中是无用的，除非应用程序能消费它们。在 AMQP 0-9-1 模型中，应用程序有两种操作方式：

* 向它们发送消息（“push” API）
* 拉取需要的消息（“pull” API）

With the "push API", applications have to indicate interest in consuming messages from a particular queue. When they do so, we say that they *register a consumer* or, simply put, *subscribe to a queue*. It is possible to have more than one consumer per queue or to register an *exclusive consumer* (excludes all other consumers from the queue while it is consuming).

在“push API”，应用程序必须表明有兴趣来自特定队列的消息。当它们这样做时，我们说它们注册了一个消费者，或者简单的说，订阅了一个队列。队列有可能有多个消费者，或者注册一个独家消费者（在消费时排除队列中的其他消费者）。

Each consumer (subscription) has an identifier called a consumer tag. It can be used to unsubscribe from messages. Consumer tags are just strings.

每个消费者（订阅）有一个叫做消费标签的标识符。它能用于取消订阅消息。消费标签只是字符串。

### Message Acknowledgements
### 消息确认

Consumer applications – applications that receive and process messages – may occasionally fail to process individual messages or will sometimes just crash. There is also the possibility of network issues causing problems. This raises a question: when should the AMQP broker remove messages from queues? The AMQP 0-9-1 specification proposes two choices:

* After broker sends a message to an application (using either basic.deliver or basic.get-ok AMQP methods).
* After the application sends back an acknowledgement (using basic.ack AMQP method).

消费应用程序，接收和处理消息的应用程序，可能偶尔处理失败单个消息或有时崩溃。网络问题也可能引发问题。这引出一个问题：AMQP 代理应该何时删除队列中的消息？AMQP 0-9-1 提供了两种选择：

* 代理向应用程序发送消息后（使用AMQP的 `basic.deliver` 或 `basic.get-ok` 方法）。
* 应用程序发回一个确认后（使用 AMQP `basic.ack` 方法）.

The former choice is called the automatic acknowledgement model, while the latter is called the explicit acknowledgement model. With the explicit model the application chooses when it is time to send an acknowledgement. It can be right after receiving a message, or after persisting it to a data store before processing, or after fully processing the message (for example, successfully fetching a Web page, processing and storing it into some persistent data store).

前一种选择称为自动确认模式，后一种称为显式确认模式。在显式确认模式，应用程序可以选择何时发送确认。它能接受到消息之后，或者处理之前存储数据之后，或完全处理消息之后（例如，成功获取网页，处理并存储到某个持久性的数据存储中）。

If a consumer dies without sending an acknowledgement the AMQP broker will redeliver it to another consumer or, if none are available at the time, the broker will wait until at least one consumer is registered for the same queue before attempting redelivery.

如果一个消费者在未发送确认的情况死掉，AMQP 代理将重新传递给其他消费者，或如果当时没有可用消费者，代理将等待直到至少有一个消费者注册相同队列，然后再尝试重新发送。

### Rejecting Messages
### 拒绝消息

When a consumer application receives a message, processing of that message may or may not succeed. An application can indicate to the broker that message processing has failed (or cannot be accomplished at the time) by rejecting a message. When rejecting a message, an application can ask the broker to discard or requeue it. When there is only one consumer on a queue, make sure you do not create infinite message delivery loops by rejecting and requeueing a message from the same consumer over and over again.

当消费者应用收到一条消息，消息的处理可能成功或不成功。应用可以通过拒绝消息向代理表明消息处理失败（或当时无法完成）。当拒绝消息时，应用程序能告诉代理丢弃或重新入队消息。当队列中只有一个消费者，确保你不会拒绝消息并重新发送消息给相同的消费者，造成无限循环的传递消息。

### Negative Acknowledgements
### 否定性确认

Messages are rejected with the `basic.reject` AMQP method. There is one limitation that `basic.reject` has: there is no way to reject multiple messages as you can do with acknowledgements. However, if you are using RabbitMQ, then there is a solution. RabbitMQ provides an AMQP 0-9-1 extension known as negative acknowledgements or nacks. For more information, please refer to the [the help page](http://www.rabbitmq.com/nack.html).

在 AMQP 方法 `basic.reject` 拒绝消息。`basic.reject `有一个限制：它没有办法像确认一样拒绝多条消息。但是，如果你使用 RabbitMQ，那么有一个解决方案，RabbitMQ 提供 一个 AMQP 0-9-1 扩展，称为否定性确认或否认。更多信息请看参考[帮助页面](http://www.rabbitmq.com/nack.html)。

### Prefetching Messages
### 预先取消息

For cases when multiple consumers share a queue, it is useful to be able to specify how many messages each consumer can be sent at once before sending the next acknowledgement. This can be used as a simple load balancing technique or to improve throughput if messages tend to be published in batches. For example, if a producing application sends messages every minute because of the nature of the work it is doing.

对于多个消费者共享队列的情况，能够在发送确认之前指定一次发送给每个消费者多少条消息。这能够作为简单的负载均衡技术，或如果消息倾向于批量发布，则可以改善吞吐量。例如，如果因为生产应用的工作性质每分钟发送消息。

Note that RabbitMQ only supports channel-level prefetch-count, not connection or size based prefetching.

注意，RabbitMQ 仅支持通道级的预取计数，不支持基于连接或大小的预取。

### Message Attributes and Payload
### 消息属性和有效负载

Messages in the AMQP model have attributes. Some attributes are so common that the AMQP 0-9-1 specification defines them and application developers do not have to think about the exact attribute name. Some examples are

* Content type
* Content encoding
* Routing key
* Delivery mode (persistent or not)
* Message priority
* Message publishing timestamp
* Expiration period
* Publisher application id

在AMQP 模型消息具有属性，一些属性是非常常见，以至于 AMQP 0-9-1 规范定义了它们，应用开发者不必考虑确切的属性名称，一些例子：

* 内容类型
* 内容编码
* 路由键
* 交付模式（持久化或不）
* 消息优先级
* 发布消息的时间戳
* 到期期限
* 应用发布者ID

Some attributes are used by AMQP brokers, but most are open to interpretation by applications that receive them. Some attributes are optional and known as headers. They are similar to X-Headers in HTTP. Message attributes are set when a message is published.

AMQP 代理使用一些属性，但大多数是通过接收它们的应用程序进行解释。有些属性是可选，称为头部。它们类似HTTP中的 X-Headers. 当一个消息发布时去设置消息属性。

AMQP messages also have a payload (the data that they carry), which AMQP brokers treat as an opaque byte array. The broker will not inspect or modify the payload. It is possible for messages to contain only attributes and no payload. It is common to use serialisation formats like JSON, Thrift, Protocol Buffers and MessagePack to serialize structured data in order to publish it as the message payload. AMQP peers typically use the "content-type" and "content-encoding" fields to communicate this information, but this is by convention only.

AMQP 消息也有一个有效负载（携带的数据），AMQP 代理将其视为不透明的字节数组。代理将不会检查或修改有效负载。消息可能只包含属性而没有有效负载。通常使用像JSON、Thrift、Protocal Buffes 和 MessagePack 去序列化结构数据，以便发布消息的有效负载。AMQP 通常使用 “content-type” 和 “content-encoding” 字段去传递这个消息，但这只是惯例而已。

Messages may be published as persistent, which makes the AMQP broker persist them to disk. If the server is restarted the system ensures that received persistent messages are not lost. Simply publishing a message to a durable exchange or the fact that the queue(s) it is routed to are durable doesn't make a message persistent: it all depends on persistence mode of the message itself. Publishing messages as persistent affects performance (just like with data stores, durability comes at a certain cost in performance).

发布的消息也可以持久化，使AMQP代理将它们保存到磁盘。如果服务重启，系统将确保收到的持久消息不会丢失。简单地将消息发布到持久交换机或它被路由到的队列是持久的，不能使消息持久化：这一切取决于消息本身的持久性模式。将消息发布为持久性会影响性能（就像数据存储，持久有一定的性能成本）。

### Message Acknowledgements
### 消息确认

Since networks are unreliable and applications fail, it is often necessary to have some kind of processing acknowledgement. Sometimes it is only necessary to acknowledge the fact that a message has been received. Sometimes acknowledgements mean that a message was validated and processed by a consumer, for example, verified as having mandatory data and persisted to a data store or indexed.

由于网络的不可靠和程序失败，消息通常需要一些处理确认。有时候只需要去确认消息真的收到了。有时候确认的意思是经过验证并消费者对消息处理完成。例如，已验证为具有强制数据并持久保存或重建索引。

This situation is very common, so AMQP 0-9-1 has a built-in feature called message acknowledgements (sometimes referred to as acks) that consumers use to confirm message delivery and/or processing. If an application crashes (the AMQP broker notices this when the connection is closed), if an acknowledgement for a message was expected but not received by the AMQP broker, the message is re-queued (and possibly immediately delivered to another consumer, if any exists).

这个情况非常常见，所以 AMQP 0-9-1 有称为消息确认的内置功能（有时称为应答），消费者使用该功能确认消息传递或处理。如果应用程序崩溃（AMQP代理在连接关闭是注意到这种情况），如果期望一个消息的确认但AMQP代理没有接收到，消息将重新入队（如果存在，可能立即传递给其他消费者）。

Having acknowledgements built into the protocol helps developers to build more robust software.

通过协议内置确认功能可以帮助开发者构建更强健的软件。

### AMQP 0-9-1 Methods
### AMQP 0-9-1 方法

AMQP 0-9-1 is structured as a number of methods. Methods are operations (like HTTP methods) and have nothing in common with methods in object-oriented programming languages. AMQP methods are grouped into classes. Classes are just logical groupings of AMQP methods. The [AMQP 0-9-1 reference](http://www.rabbitmq.com/amqp-0-9-1-reference.html) has full details of all the AMQP methods.

AMQP 0-9-1 是由多种方法构成。方法是操作（像HTTP方法），和面向对象语言的方法没有共同之处。AMQP 方法是分组到类中。类只是AMQP方法的逻辑分组。这个 [AMQP 0-9-1 参考](http://www.rabbitmq.com/amqp-0-9-1-reference.html)是所有 AMQP 方法的详细说明。

Let us take a look at the *exchange class*, a group of methods related to operations on exchanges. It includes the following operations:

* exchange.declare
* exchange.declare-ok
* exchange.delete
* exchange.delete-ok

让我们看看*交换机分类*，这是一组说明交换机操作的方法。它包含如下操作：

* exchange.declare
* exchange.declare-ok
* exchange.delete
* exchange.delete-ok

(note that the RabbitMQ site reference also includes RabbitMQ-specific extensions to the exchange class that we will not discuss in this guide).

（注意，RabbitMQ 网站参考也包含RabbitMQ特定去扩展交换类型，我们将不再本指南中讨论）。

The operations above form logical pairs: `exchange.declare` and `exchange.declare-ok`, `exchange.delete` and `exchange.delete-ok`. These operations are "requests" (sent by clients) and "responses" (sent by brokers in response to the aforementioned "requests").

上面的操作形成逻辑对：`exchange.declare` 和 `exchange.declare-ok`，`exchange.delete` 和 `exchange.delete-ok`。这些操作是“请求”（由客户端发送）和“响应”（由代理响应上述的“请求”）。

As an example, the client asks the broker to declare a new exchange using the `exchange.declare` method:

一个例子，客户端要求代理使用 `exchange.declare` 方法声明一个新的交换机：

![exchange-declare](http://pa1so03xn.bkt.clouddn.com/exchange-declare.png)


As shown on the diagram above, `exchange.declare` carries several parameters. They enable the client to specify exchange name, type, durability flag and so on.

如上图所示，`exchange.declare`带有几个参数，它们使客户端能指明交换机的名称，类型，持久化标志等。

If the operation succeeds, the broker responds with the `exchange.declare-ok` method:

如果操作成功，代理回复`exchange.declare-ok`方法：

![exchange-declare-ok](http://pa1so03xn.bkt.clouddn.com/exchange-declare-ok.png)


`exchange.declare-ok`does not carry any parameters except for the channel number (channels will be described later in this guide).

`exchange.declare-ok`除了通道号不带有任何参数（通道将在本指南后面介绍）。

The sequence of events is very similar for another method pair on the AMQP queue class: queue.declare and queue.declare-ok:

对于 AMQP 队列类中的另一个方法对，事件顺序也非常相似：

![queue-declare](http://pa1so03xn.bkt.clouddn.com/queue-declare.png)

![queue-declare-ok](http://pa1so03xn.bkt.clouddn.com/queue-declare-ok.png)


Not all AMQP methods have counterparts. Some (`basic.publish` being the most widely used one) do not have corresponding "response" methods and some others (`basic.get`, for example) have more than one possible "response".

不是所有的 AMQP 方法都是成对的。有些（`basic.publish`是使用最广泛的）没有相应的“response”方法，或其他一些（例如：`basic.get`）可能有多个“response”。

### Connections
### 连接

AMQP connections are typically long-lived. AMQP is an application level protocol that uses TCP for reliable delivery. AMQP connections use authentication and can be protected using TLS (SSL). When an application no longer needs to be connected to an AMQP broker, it should gracefully close the AMQP connection instead of abruptly closing the underlying TCP connection.

AMQP 连接通常是长期存活的。AMQP 是一种使用 TCP 进行可靠传输的应用层协议。AMQP 连接使用 TLS(SSL) 保护进行身份验证。当应用不需要长时间连接 AMQP 代理时，它应该正常的关闭 AMQP 连接，而不是突然关闭底层的 TCP 连接。

### Channels
### 通道

Some applications need multiple connections to an AMQP broker. However, it is undesirable to keep many TCP connections open at the same time because doing so consumes system resources and makes it more difficult to configure firewalls. AMQP 0-9-1 connections are multiplexed with channels that can be thought of as "lightweight connections that share a single TCP connection".

有些应用程序需要连接多个 AMQP 代理。但是，不希望同时打开许多 TCP 连接，因为这样做会消耗系统资源并使配置防火墙更加困难。AMQP 0-9-1 连接是*通道*多路复用，可以看做“共享单个 TCP 连接的轻量级连接”。

For applications that use multiple threads/processes for processing, it is very common to open a new channel per thread/process and not share channels between them.

对于使用多线程/进程进行处理的应用程序，每个线程/进程常见的是打开一个新的通道，而不是在它们之间共享通道。

Communication on a particular channel is completely separate from communication on another channel, therefore every AMQP method also carries a channel number that clients use to figure out which channel the method is for (and thus, which event handler needs to be invoked, for example).

特定通道上的通信和其他通道上的通信是完全分开，因此每个AMQP 方法也带有通道号，客户端使用通道号来确定方法所对应的通道（例如，需要调用的某个事件处理程序）。

### Virtual Hosts
### 虚拟主机

To make it possible for a single broker to host multiple isolated "environments" (groups of users, exchanges, queues and so on), AMQP includes the concept of virtual hosts (vhosts). They are similar to virtual hosts used by many popular Web servers and provide completely isolated environments in which AMQP entities live. AMQP clients specify what vhosts they want to use during AMQP connection negotiation.

为了使单个代理可以托管多个隔离“环境”（用户组，交换机，队列等），AMQP 包含虚拟主机（vhosts）概念。它们类似大多流行Web服务器使用的虚拟主机，并为AMQP实体提供完整的隔离环境。AMQP 客户端指定在 AMQP 连接协商期间使用的虚拟主机。

### AMQP is Extensible
### AMQP 是可扩展的

AMQP 0-9-1 has several extension points:
AMQP 0-9-1 有几个可扩展点：

* Custom exchange types let developers implement routing schemes that exchange types provided out-of-the-box do not cover well, for example, geodata-based routing.
* Declaration of exchanges and queues can include additional attributes that the broker can use. For example, [per-queue message TTL](http://www.rabbitmq.com/ttl.html) in RabbitMQ is implemented this way.
* Broker-specific extensions to the protocol. See, for example, [extensions that RabbitMQ implements](http://www.rabbitmq.com/extensions.html).
* [New AMQP 0-9-1 method classes](http://www.rabbitmq.com/amqp-0-9-1-quickref.html#class.confirm) can be introduced.
* Brokers can be extended with [additional plugins](http://www.rabbitmq.com/plugins.html), for example, the [RabbitMQ management](http://www.rabbitmq.com/management.html) frontend and HTTP API are implemented as a plugin.

* 自定义交换机类型让开发者实现路由方案，交换机类型不能很好的提供开箱即用，例如，地理数据路由。
* 声明交换机和队列能包含代理使用的附加属性。例如，在 RabbitMQ 中用这种方法实现[队列消息 TTL](http://www.rabbitmq.com/ttl.html)。
* 特定于代理的扩展协议。例如，请参阅[RabbitMQ实现的扩展](http://www.rabbitmq.com/extensions.html)。
* 可以引入新的 [AMQP 0-9-1 方法类](http://www.rabbitmq.com/amqp-0-9-1-quickref.html#class.confirm)。
* 代理可以扩展的[附加插件](http://www.rabbitmq.com/plugins.html)，例如，RabbitMQ 的[前台管理](http://www.rabbitmq.com/management.html)和HTTP API 作为插件实现。

These features make the AMQP 0-9-1 Model even more flexible and applicable to a very broad range of problems.

这些功能使 AMQP 0-9-1 模型更加灵活，适用更广泛的问题。

### AMQP 0-9-1 Clients Ecosystem
### AMQP 0-9-1 客户端生态环境

There are many[AMQP 0-9-1 clients](http://www.rabbitmq.com/devtools.html) for many popular programming languages and platforms. Some of them follow AMQP terminology closely and only provide implementation of AMQP methods. Some others have additional features, convenience methods and abstractions. Some of the clients are asynchronous (non-blocking), some are synchronous (blocking), some support both models. Some clients support vendor-specific extensions (for example, RabbitMQ-specific extensions).

许多流行的编程语言和平台都有 [AMQP 0-9-1 客户端](http://www.rabbitmq.com/devtools.html)。其中一些严格遵循 AMQP 术语和只提供 AMQP 方法的实现。其他一些有更多功能，有用方法和抽象。一些客户端是异步（非阻塞），一些是同步（阻塞），一些支持两种模型。一些客户端支持特定扩展（例如，RabbitMQ 特定扩展）。

Because one of the main AMQP goals is interoperability, it is a good idea for developers to understand protocol operations and not limit themselves to terminology of a particular client library. This way communicating with developers using different libraries will be significantly easier.

因为 AMQP 的主要目标之一是互操作性，因此开发者理解协议操作并不限制于特定客户端库的术语是很好的。通过这种方式和使用不同库的开发者交流将更加容易。



