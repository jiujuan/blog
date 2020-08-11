## 一、Micro

### 架构图
![go-micro-img-one](../images/go-micro-arch-01.png)

 Micro 是一个平台，它使用可插拔的基础和强定义的 api 来解决这些问题。可插入任何堆栈或云.

 运行时由以下功能组成:

- **api gateway**: api 网关。使用服务发现具有动态请求路由的单个入口点. API 网关允许您在后端构建可扩展的微服务体系结构，并在前端合并公共 api. micro api 通过发现和可插拔处理程序提供强大的路由，为 http, grpc, Websocket, 发布事件等提供服务.
- **broker**: 允许异步消息的消息代理。微服务是事件驱动的体系结构，应该作为一等公民提供消息传递。通知其他服务的事件，而无需担心响应.
- **network**: 通过微网络服务构建多云网络。只需跨任何环境连接网络服务，创建单个平面网络即可全局路由. Micro 的网络根据每个数据中心中的本地注册表动态构建路由，确保根据本地设置路由查询.
- **new**: 服务模板生成器。创建新的服务模板以快速入门. Micro 提供用于编写微服务的预定义模板。始终以相同的方式启动，构建相同的服务以提高工作效率.
- **proxy**: 建立在 Go Micro 上的透明服务代理。将服务发现，负载平衡，容错，消息编码，中间件，监视等代理路由到某个位置。独立运行它或与服务一起运行.
- **registry**: 注册表提供服务发现以查找其他服务，存储功能丰富的元数据和终结点信息。它是一个服务资源管理器，允许您在运行时集中和动态地存储此信息.
- **store**: 有状态是任何系统的必然需求。我们提供密钥值存储，提供简单的状态存储，可在服务之间共享或长期持有 以保持微服务无状态和水平可扩展.
- **web**: Web 仪表板允许您浏览服务，描述其终结点，请求和响应格式，甚至直接查询它们。仪表板还包括内置 CLI 的体验，适用于希望动态进入终端的开发人员.

此外，Micro还提供了Go开发框架：
- **go-micro**: 利用强大的 [Go Micro](https://github.com/micro/go-micro) 框架轻松，快速地开发微服务. Go Micro 抽象了分布式系统的复杂性，并提供了更简单的抽象来构建高度可扩展的微服务.

### 什么是Micro

Micro 使开发人员能够构建、共享和协作式的分布式系统：
- Micro 有一个 [运行时](https://github.com/micro/micro)，可以管理你的微服务

- Micro 有一个 [go-micro 框架](https://github.com/micro/go-micro)，可以编写服务

- Micro 有一个 [社区](https://slack.micro.mu/)，可以讨论和解答Micro的各种问题

Micro是一个多工具集合， 有micro这个管理工具、go-micro框架 和 go-plugins插件 等各种工具和框架，以帮助人们更好的进行微服务开发。

- **框架** - go-micro框架，编写微服务的 Go 框架；服务发现，远程过程调用，发布 / 订阅等.
- **运行时** - 微服务运行时环境；用于管理服务包括auth，config，discovery，networking的运行时环境。.
- **插件** - 框架和运行时的插件，包括 etcd, kubernets, nats, grpc, 等.
- **客户端**：多语言客户端
可以在 [**github.com/micro**](https://github.com/micro) 上找到所有的工具

使用 [micro](https://github.com/micro/micro) 工具包通过 cli, web ui, slack 或 api 网关访问微服务.

Micro 由开源库与工具组成，旨在辅助微服务开发。

- [micro](https://github.com/micro/micro) - 微服务工具集包含传统的入口点（entry point）；API 网关、CLI、Slack Bot、代理及 Web UI。还有其它相关的库和服务可以参考 github.com/micro。
  - API可以作为单个http入口点
   - Web仪表盘用于可视化服务
   - 用于命令行访问的 CLI
   - 全新魔板生成代码可快速启动
   - 通过 Slack 或 HipChat 查询的机器人供
- [go-micro](https://github.com/micro/go-micro) - 基于 Go 语言的可插拔 RPC 微服务开发框架；包含服务发现、RPC 客户/服务端、广播/订阅机制等等。
  - 抽象了分布式系统的元素
  - 服务发现、远程调用-RPC，发布订阅等功能
  - 超时、重试和负载均衡等功能
  - 通过包装器还可以扩展功能
  - 可插拔的接口设计
- [go-plugins](https://github.com/micro/go-plugins) - go-micro 的插件库，有 etcd、kubernetes、nats、rabbitmq、grpc 等等。
  - 用于 go-micro/micro 的插件
   - 包括一些流行的后端技术，grpc，kubernetes，kafka等
   - 还有一些经过生成检验过的插件

### 如何使用Micro？

1. 可以通过 [micro](https://github.com/micro/micro) 工具包访问它们.
2. 可以使用 [go-micro](https://github.com/micro/go-micro) 编写服务.

#### 使用工具包运行服务

[工具包命令参考文档](https://learnku.com/docs/go-micro/2.x/getting-started.html/8460)，这个是中文翻译版。

[英文 docs](https://github.com/micro/docs/blob/master/getting-started/README.md)

在bash里运行命令 `micro server`， 这个命令来启动micro服务器，如果顺利的话，会输出很多初始化的日志。
与此服务器交互，可以用CLI，交互命令为：`micro env set server`

然后可以查看运行的服务：`micro list services`
```shell
# micro list services
go.micro.api
go.micro.auth
go.micro.bot
go.micro.broker
go.micro.config
go.micro.debug
go.micro.network
go.micro.proxy
go.micro.registry
go.micro.router
go.micro.runtime
go.micro.server
go.micro.web
```

可以去 [github.com/micro/services](https//github.com/micro/services) 看看, 我们看到一堆由 mirco 作者编写的服务。你可以查看文档，然后用命令试试其中的一些服务。

### API，Web和SRV服务之间的区别是什么？
先看一张图：
![go-micro-img-two](../images/go-micro-api-srv-web-02.png)

作为 micro 工具包的一部分, 我们尝试通过分离 API, Web 仪表板和后端服务 (SRV） 的关注点, 为可扩展体系结构定义一组设计模式.

可以这么理解：API就是前端的访问，SRV就是后端提供的服务， 而 Web 仪表盘就是能查看这些，可视化。

- **API服务**
API服务是单独的一个服务层，用于为http/json API提供服务。API服务由micro api提供。
可以用go-micro编写标准的服务，使用命名空间go.micro.api在逻辑上进行分离。

- **Web服务**
Web服务又默认命名空间go.micro.web提供服务。Micro Web是反向代理，将根据服务解析路径到HTTP请求转发到相应的Web应用。

- **SRV服务**
SRV服务是标准的RPC服务，是你通常编写的服务类型。通常称他们为RPC服务或后端服务，他们是后端体系的一部分，不是面向公众的。默认情况下，使用命名空间 go.micro.srv 进行操作，但你应该使用域 com.example.srv。

## 二、Go-Micro

### 什么是Go-Micro
go-micro是一个微服务开发的框架，是一个插件式的RPC框架。它用于分布式系统开发。这个插件抽象出了分布式系统的细节。
![three](../images/go-micro-arch-program-03.png)

- 最顶层的 Service 接口是构建服务的主要组件，它把底层的各个包需要实现的接口，做了一次封装，包含了一系列用于初始化 Service 和 Client 的方法，使我们可以很简单的创建一个 RPC 服务；
- Client 是请求服务的接口，从 Registry 中获取 Server 信息，然后封装了 Transport 和 Codec 进行 RPC 调用，也封装了 Brocker 进行消息发布，默认通过 RPC 协议进行通信，也可以基于 HTTP 或 gRPC；

- Server 是监听服务调用的接口，也将以接收 Broker 推送过来的消息，需要向 Registry 注册自己的存在与否，以便客户端发起请求，和 Client 一样，默认基于 RPC 协议通信，也可以替换为 HTTP 或 gRPC；
- Broker 是消息发布和订阅的接口，默认实现是基于 HTTP，在生产环境可以替换为 Kafka、RabbitMQ 等其他组件实现；
- Codec 用于解决传输过程中的编码和解码，默认实现是 protobuf，也可以替换成 json、mercury 等；
- Registry 用于实现服务的注册和发现，当有新的 Service 发布时，需要向 Registry 注册，然后 Registry 通知客户端进行更新，Go Micro 默认基于 consul 实现服务注册与发现，当然，也可以替换成 etcd、zookeeper、kubernetes 等；
- Selector 是客户端级别的负载均衡，当有客户端向服务端发送请求时，Selector 根据不同的算法从 Registery 的主机列表中得到可用的 Service 节点进行通信。目前的实现有循环算法和随机算法，默认使用随机算法，另外，Selector 还有缓存机制，默认是本地缓存，还支持 label、blacklist 等方式；
- Transport 是服务之间通信的接口，也就是服务发送和接收的最终实现方式，默认使用 HTTP 同步通信，也可以支持 TCP、UDP、NATS、gRPC 等其他方式。

Go Micro 官方创建了一个 [Plugins](https://github.com/micro/go-plugins) 仓库，用于维护 Go Micro 核心接口支持的可替换插件：

| 接口      | 支持组件                             |
| --------- | ------------------------------------ |
| Broker    | NATS、NSQ、RabbitMQ、Kafka、Redis 等 |
| Client    | gRPC、HTTP                           |
| Codec     | BSON、Mercury 等                     |
| Registry  | Etcd、NATS、Kubernetes、Eureka 等    |
| Selector  | Label、Blacklist、Static 等          |
| Server    | gRPC、HTTP                           |
| Transport | NATS、gPRC、RabbitMQ、TCP、UDP       |
| Wrapper   | 中间件：熔断、限流、追踪、监控       |

各个组件接口之间的关系可以通过下图串联：
![four-image](../images/go-micro-srv-componets-04.png)


## Go-Micro 学习资源
- [micro](https://github.com/micro/micro)
- [go-micro](https://github.com/micro/go-micro)
- [go-plugins](https://github.com/micro/go-plugins)
- [docs](https://github.com/micro/docs)
- [site](https://m3o.dev/)
- [go-micro 2.X中文文档](https://learnku.com/docs/go-micro/2.x)
- [go-micro一些资源](https://learnku.com/docs/go-micro/2.x/resources/8459)
- [awesome-micro](https://github.com/micro-community/awesome-micro)
- [micro-in-cn](https://github.com/micro-in-cn)

## 参考
- [go-micro 2.x 中文文档](https://learnku.com/docs/go-micro/2.x)
- [micro](https://github.com/hb-go/micro)