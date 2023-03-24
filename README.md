更多文章请看：[我的博客](https://www.cnblogs.com/jiujuan/)： https://www.cnblogs.com/jiujuan/

## 目录

- [云原生和kubernetes](#云原生和kubernetes)
- [微服务](#微服务)
  - [微服务架构思考系列](#微服务架构思考系列)
  - [Go 微服务实战系列](#go-kratos-微服务实战系列)
- [Redis原理学习](#redis原理学习)
- [MySQL](#mysql)
- [架构](#架构)
- [gRPC 学习](#grpc-学习)
- [深入理解golang](#深入理解golang)
- [操作系统和其他](#操作系统和其他)
- [golang常用库包使用](#golang常用库包使用)
- [Go 语言并发编程](#go-语言并发编程)
- [Go 性能优化分析](#go-性能优化分析)
- [Gin Web 框架学习系列](#gin-web-框架学习系列)
- [Go 开发规范(软件工程)](#go-开发规范软件工程)
- [消息队列](#消息队列)
- [监控](#监控)
- [管理-产品相关](#管理-产品相关)
  - [管理](#管理)
  - [产品](#产品)
- [学习成长](#学习成长)
- [java](#java)
- [网络协议](#协议)
- [九卷读书](#九卷读书)


## 云原生和kubernetes

- [云原生学习101-云原生概览](https://github.com/jiujuan/zenblog/blob/master/content/cloud-native-learning-intro-101-1.md)
- [云原生概览](https://www.cnblogs.com/jiujuan/p/16158019.html)

- [Docker常用命令总结](https://www.cnblogs.com/jiujuan/p/13758351.html)

- [kubernetes(k8s)基础学习-kubernetes是什么？有什么用？](https://www.cnblogs.com/jiujuan/p/17107088.html)

## 微服务

### 微服务架构思考系列
- [微服务架构学习与思考(01)：什么是微服务？微服务的优势和劣势](https://www.cnblogs.com/jiujuan/p/13280473.html)
- [微服务架构学习与思考(02)：微服务实施前有哪些问题需要思考？](https://www.cnblogs.com/jiujuan/p/13284412.html)
- [微服务架构学习与思考(03)：微服务总体架构图解](https://www.cnblogs.com/jiujuan/p/13295147.html)
- [微服务架构学习与思考(04)：微服务技术体系](https://www.cnblogs.com/jiujuan/p/13301055.html)
- [微服务架构学习与思考(05)：微服务架构适用场景分析](https://www.cnblogs.com/jiujuan/p/13762969.html)
- [微服务架构学习与思考(06)：如何构建微服务？](https://www.cnblogs.com/jiujuan/p/14070123.html)
- [微服务架构学习与思考(07)：企业团队组织架构如何变革？](https://www.cnblogs.com/jiujuan/p/14321594.html)
- [微服务架构学习与思考(08)：服务注册中心（服务注册与服务发现）](https://www.cnblogs.com/jiujuan/p/15087196.html)
- [微服务架构学习与思考(09)：分布式链路追踪系统-dapper论文学习](https://www.cnblogs.com/jiujuan/p/16097314.html)
- [微服务架构学习与思考(10)：微服务网关和开源 API 网关01-以 Nginx 为基础的 API 网关详细介绍](https://www.cnblogs.com/jiujuan/p/16810290.html)
- [微服务架构学习与思考(11)：开源 API 网关02-以 Java 为基础的 API 网关详细介绍](https://www.cnblogs.com/jiujuan/p/16814545.html)
- [微服务架构学习与思考(12)：从单体架构到微服务架构的演进历程](https://www.cnblogs.com/jiujuan/p/17066590.html)

- [小公司需要使用微服务架构吗？](https://www.cnblogs.com/jiujuan/p/17116605.html)

### 服务治理
- [服务治理：常用限流算法总结](https://www.cnblogs.com/jiujuan/p/16264022.html)
- [服务治理：几种开源限流算法库/应用软件介绍和使用](https://www.cnblogs.com/jiujuan/p/16283141.html)

### go-kratos 微服务实战系列
- [Go微服务框架go-kratos实战学习01：quickstart 快速创建项目 ](https://www.cnblogs.com/jiujuan/p/16322725.html)
- [Go微服务框架go-kratos实战学习02：proto 代码生成和项目代码编写步骤](https://www.cnblogs.com/jiujuan/p/16331967.html)
- [Go微服务框架go-kratos实战学习03：使用 gorm 实现增删改查操作](https://www.cnblogs.com/jiujuan/p/16338305.html)
- [Go微服务框架go-kratos实战学习04：服务注册和服务发现(etcd作为注册中心)](https://www.cnblogs.com/jiujuan/p/16341183.html)
- [Go微服务框架go-kratos实战学习05：分布式链路追踪 OpenTelemetry 使用](https://www.cnblogs.com/jiujuan/p/16349519.html)
- [Go微服务框架go-kratos实战学习06：配置使用-file和nacos配置中心](https://www.cnblogs.com/jiujuan/p/16498072.html)
- [Go微服务框架go-kratos实战学习07：consul 作为服务注册和发现中心](https://www.cnblogs.com/jiujuan/p/17196383.html)

> [go-kratos实战代码地址](https://github.com/jiujuan/go-kratos-demos)

- [golang微服务实践：分布式链路追踪系统-jaeger安装与简单使用](https://www.cnblogs.com/jiujuan/p/13235748.html)
- [golang微服务实践：服务注册与服务发现 - Etcd的使用](https://www.cnblogs.com/jiujuan/p/13200898.html)

### go-kratos 分析
- [Golang微服务框架go-kratos分析：框架架构分析](https://www.cnblogs.com/jiujuan/p/16845565.html)

### go-micro 微服务实践

- [golang微服务框架go-micro学习(01)-Micro微服务框架整体介绍](https://github.com/jiujuan/zenblog/blob/master/content/go-micro-learning-part-one.md)

## Redis原理学习
- [Redis原理学习：Redis主体流程分析](https://www.cnblogs.com/jiujuan/p/12884938.html)
- [Redis服务端事件处理流程分析](https://www.cnblogs.com/jiujuan/p/16723573.html)
- [Redis原理再学习01：数据结构-跳跃表skiplist](https://www.cnblogs.com/jiujuan/p/12884824.html)
- [Redis原理再学习02：数据结构-动态字符串sds](https://www.cnblogs.com/jiujuan/p/15828302.html)
- [Redis原理再学习03：数据结构-链表 list](https://www.cnblogs.com/jiujuan/p/15839269.html)
- [Redis原理再学习04：数据结构-哈希表hash表(dict字典)](https://www.cnblogs.com/jiujuan/p/15944061.html)
- [Redis原理再学习05：数据结构-整数集合intset](https://www.cnblogs.com/jiujuan/p/15993249.html)
- [Redis高可用之主从复制原理演进分析](https://www.cnblogs.com/jiujuan/p/16784964.html)
- [Redis 缓存过期删除/淘汰策略分析](https://www.cnblogs.com/jiujuan/p/15765214.html)

- [认识Redis持久化RDB、AOF和混合持久化](https://www.cnblogs.com/jiujuan/p/10877805.html)
- [Redis复制原理](https://www.cnblogs.com/jiujuan/p/11278486.html)
- [redis高可用集群-redis cluster(cluster集群)简介和配置（3）](https://www.cnblogs.com/jiujuan/p/10409600.html)
- [redis高可用集群-sentinel哨兵高可用配置（2）](https://www.cnblogs.com/jiujuan/p/9091337.html)
- [redis高可用集群-redis主从复制配置（1）](https://www.cnblogs.com/jiujuan/p/9071003.html)


- [Redis 缓存常见问题：缓存穿透、缓存击穿和缓存雪崩](https://www.cnblogs.com/jiujuan/p/10441214.html)

- [阿里云Redis开发规范](https://www.cnblogs.com/jiujuan/p/10441225.html)

## MySQL
- [如果有人问你数据库的原理，叫他看这篇文章](https://www.cnblogs.com/jiujuan/p/10676119.html)

- [MySQL InnoDB存储引擎大观](https://www.cnblogs.com/jiujuan/p/11336049.html)
- [五大常见的MySQL高可用方案](https://www.cnblogs.com/jiujuan/p/10907854.html)

## 架构
- [了解企业架构EA(Enterprise Architecture)](https://www.cnblogs.com/jiujuan/p/16871889.html)

- [分层架构设计总结](https://www.cnblogs.com/jiujuan/p/16880756.html)

- [《恰如其分的软件架构》笔记摘要](https://www.cnblogs.com/jiujuan/p/13594006.html)

- [RESTful API 介绍，设计](https://www.cnblogs.com/jiujuan/p/12791574.html)

## gRPC 学习

### gRPC学习

- [golang gRPC学习(01)：gRPC介绍](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-introduce-1.md)
- [Golang gRPC学习(02): 编写helloworld服务](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-write-helloworld-2.md)
- [Golang gRPC学习(03): grpc官方示例程序route_guide简析](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-explain-route-guide-3.md)
- [Golang gRPC学习(04): Deadlines超时限制](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-deadlines-timeout-4.md)
- [Golang gRPC学习(05): retry重试](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-retry-5.md)
- [Golang gRPC学习(06): TLS/SSL认证](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-tls-ssl-6.md)
- [Golang gRPC学习(07): interceptor拦截器](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-%20interceptor-7.md)
- [Golang gRPC学习(08): go-grpc-middleware](https://github.com/jiujuan/zenblog/blob/master/content/golang-grpc-middleware-8.md)



### gRPC示例代码地址

gRPC学习示例[代码地址](https://github.com/jiujuan/grpc-tutorial)



## 操作系统和其他
- [到底什么是锁？](./content/what-is-lock-1.md)
- [基于Redis的分布式锁真的安全吗？](https://www.cnblogs.com/jiujuan/p/10595838.html)

- [etcd实现分布式锁分析](https://www.cnblogs.com/jiujuan/p/12147809.html)
- [consul实现分布式锁](https://www.cnblogs.com/jiujuan/p/10527786.html)

- [服务端高性能网络IO编程模型简析](https://www.cnblogs.com/jiujuan/p/16586900.html)
- [linux和unix中的IO模型总结](https://www.cnblogs.com/jiujuan/p/16564610.html)
- [对进程、线程和协程的理解以及它们的区别](https://www.cnblogs.com/jiujuan/p/16193142.html)

- [Linux内存管理](https://www.cnblogs.com/jiujuan/p/10712197.html)
- [Linux进程的虚拟内存](https://www.cnblogs.com/jiujuan/p/12054748.html)
- [Linux内存：物理内存管理概述](https://www.cnblogs.com/jiujuan/p/12054692.html)
- [Linux进程: task_struct结构体成员](https://www.cnblogs.com/jiujuan/p/11715853.html)
- [Linux进程：管理和调度](https://www.cnblogs.com/jiujuan/p/12040884.html)
- [线程，进程和并发](https://www.cnblogs.com/jiujuan/p/8911898.html)

## 深入理解golang
- [深入理解Go语言(01): interface源码分析](https://www.cnblogs.com/jiujuan/p/12653806.html)
- [深入理解Go语言(02): channels - kavya Joshi](https://www.cnblogs.com/jiujuan/p/12026551.html)
- [深入理解Go语言(03)：scheduler调度器 - 基本介绍](https://www.cnblogs.com/jiujuan/p/12735559.html)
- [深入理解Go语言(04)：scheduler调度器-GPM源码分析](https://www.cnblogs.com/jiujuan/p/12977832.html)
- [深入理解Go语言(05)：sync.map原理分析](https://www.cnblogs.com/jiujuan/p/13365901.html)
- [深入理解Go语言(06)：Context原理分析](https://www.cnblogs.com/jiujuan/p/13795723.html)
- [TCMalloc 内存分配原理简析](https://www.cnblogs.com/jiujuan/p/13922551.html)
- [深入理解Go语言(07)：内存分配原理](https://www.cnblogs.com/jiujuan/p/13922551.html)
- [深入理解Go语言(08)：sync.WaitGroup源码分析](https://www.cnblogs.com/jiujuan/p/16735012.html)

- [Golang 汇编asm语言基础学习](https://www.cnblogs.com/jiujuan/p/16555192.html)

- [go http server 编程实践及源码分析](https://www.cnblogs.com/jiujuan/p/11762587.html)

## golang常用库包使用
- [golang常用库：字段参数验证库-validator使用](https://www.cnblogs.com/jiujuan/p/13823864.html)
- [golang常用库：操作数据库的orm框架-gorm基本使用](https://www.cnblogs.com/jiujuan/p/12676195.html)
- [golang常用库：gorilla/mux-http路由库使用](https://www.cnblogs.com/jiujuan/p/12768907.html)
- [golang常用库：配置文件解析库/管理工具-viper使用](https://www.cnblogs.com/jiujuan/p/13799976.html)
- [golang常用库包：http和API客户端请求库-go-resty](https://www.cnblogs.com/jiujuan/p/14583605.html)
- [golang常用库包：cli命令行/应用程序生成工具-cobra使用](https://www.cnblogs.com/jiujuan/p/15487918.html)
- [golang常用库包：日志记录库-logrus使用](https://www.cnblogs.com/jiujuan/p/15542743.html)
- [golang常用库包：Go依赖注入(DI)工具-wire使用](https://www.cnblogs.com/jiujuan/p/16136633.html)
- [golang常用库包：redis操作库go-redis使用(01)-Redis数据类型简介和连接Redis的几种方式](https://www.cnblogs.com/jiujuan/p/17207166.html)
- [golang常用库包：redis操作库go-redis使用(02)-Redis5种基本数据类型操作](https://www.cnblogs.com/jiujuan/p/17215125.html)
- [golang常用库包：redis操作库go-redis使用(03)-高级数据结构和其它特性](https://www.cnblogs.com/jiujuan/p/17231723.html)

- [Go package(3)：io包介绍和使用](https://www.cnblogs.com/jiujuan/p/14005731.html)

- [golang 中 channel 的详细使用、使用注意事项及死锁分析](https://www.cnblogs.com/jiujuan/p/16723573.html)
- [Go 中的反射 reflect 介绍和基本使用](https://www.cnblogs.com/jiujuan/p/17142703.html)

## Go 语言并发编程
- [Go语言并发编程(1)：对多进程、多线程、协程和并发、并行的理解](https://www.cnblogs.com/jiujuan/p/17151595.html)
- [Go语言并发编程(2)：channel 通道介绍和使用](https://www.cnblogs.com/jiujuan/p/17248549.html)
- [Go语言并发编程(3)：sync包介绍和使用(上)-Mutex,RWMutex,WaitGroup,sync.Map](https://www.cnblogs.com/jiujuan/p/17248551.html)
- [Go语言并发编程(4)：sync包介绍和使用(下)-Once,Pool,Cond](https://www.cnblogs.com/jiujuan/p/17248552.html)

## Go 性能优化分析
- [golang 性能优化分析工具 pprof (上) - 基础使用介绍](https://www.cnblogs.com/jiujuan/p/14588185.html)
- [golang 性能优化分析工具 pprof（下）- web 服务分析](https://www.cnblogs.com/jiujuan/p/14598141.html)
- [golang 性能优化分析：benchmark 结合 pprof](https://www.cnblogs.com/jiujuan/p/14604609.html)

## Gin Web 框架学习系列
- [01-quickstart](https://github.com/jiujuan/gin-tutorial/blob/master/01quickstart/main.go)
- [02-parameter](https://github.com/jiujuan/gin-tutorial/tree/master/02parameter)
- [03-route](https://github.com/jiujuan/gin-tutorial/tree/master/03route)
- [04-middleware](https://github.com/jiujuan/gin-tutorial/tree/master/04middleware)
- [05-log](https://github.com/jiujuan/gin-tutorial/tree/master/05log)
- [06-logrus](https://github.com/jiujuan/gin-tutorial/tree/master/06logrus)
- [07-bind](https://github.com/jiujuan/gin-tutorial/tree/master/07bind)
- [08-validate](https://github.com/jiujuan/gin-tutorial/tree/master/08validate)
- [09-restful](https://github.com/jiujuan/gin-tutorial/tree/master/09gin-gorm-restful)
- [10-blog](https://github.com/jiujuan/gin-tutorial/tree/master/10gin-blog)
- [11-jwt](https://github.com/jiujuan/gin-tutorial/tree/master/11gin-jwt/demo1)
- [12-shutdown/gracefull](https://github.com/jiujuan/gin-tutorial/tree/master/12shutdown/gracefull)
- [13-livereload](https://github.com/jiujuan/gin-tutorial/tree/master/13livereload)
- [14-swagger](https://github.com/jiujuan/gin-tutorial/tree/master/14swagger)
- [15-vue-gorm](https://github.com/jiujuan/gin-tutorial/tree/master/15vue-gorm)
- [16-hystrix-go](https://github.com/jiujuan/gin-tutorial/tree/master/16hystrix-go) hystrix-go和gin结合使用的demo

---

- [一步一步分析Gin框架路由源码及radix tree基数树](https://www.cnblogs.com/jiujuan/p/15973485.html)

- [golang程序设计：Go middleware中间件以及Gin 中间件分析](https://www.cnblogs.com/jiujuan/p/13069995.html)

## Go 开发规范(软件工程)
- [Go 项目文件布局](https://www.cnblogs.com/jiujuan/p/16805835.html)
- [Uber Go 编码规范](https://www.cnblogs.com/jiujuan/p/15568018.html)

## 消息队列
- [消息队列常见问题分析](https://www.cnblogs.com/jiujuan/p/13732880.html)

- [kafka学习笔记01-kafka简介和架构介绍](https://www.cnblogs.com/jiujuan/p/15024642.html)
- [kafka学习笔记02-kafka消息存储](https://www.cnblogs.com/jiujuan/p/15024666.html)
- [kafka学习笔记03-消息生产者producer](https://www.cnblogs.com/jiujuan/p/15055979.html)

## 监控
- [Prometheus监控+Grafana+Alertmanager告警安装使用 (图文详解)](https://www.cnblogs.com/jiujuan/p/13262380.html)

## 管理-产品相关

### 管理
- [聊一聊向上管理](https://www.cnblogs.com/jiujuan/p/16611584.html)
- [技术管理的一些理念](https://www.cnblogs.com/jiujuan/p/14682355.html)
- [技术人怎么“打通”产品业务？](https://www.cnblogs.com/jiujuan/p/13467196.html)

- [碎碎念软件研发01：敏捷简史和几种软件开发模型](https://www.cnblogs.com/jiujuan/p/16294276.html)
- [碎碎念软件研发02：敏捷之Scrum](https://www.cnblogs.com/jiujuan/p/16319726.html)

- [如何提高团队管理能力？](https://www.cnblogs.com/jiujuan/p/9004708.html)
- [如何多角度思考问题？](https://www.cnblogs.com/jiujuan/p/15717510.html)
- [聊一聊结构化思维](https://www.cnblogs.com/jiujuan/p/13364052.html)
- [反思：进步的“魔镜”](https://www.cnblogs.com/jiujuan/p/11995383.html)
- [工作中常用的方法(思维模型)](https://www.cnblogs.com/jiujuan/p/13394703.html)

- [技术人沟通中的几个常见问题分析和解答](https://www.cnblogs.com/jiujuan/p/13842338.html)

- [用价值链分析法: 分析软件开发的价值定位、赋能及创业杂感](https://www.cnblogs.com/jiujuan/p/13676593.html)

- [业务 产品 技术的一点看法](https://www.cnblogs.com/jiujuan/p/11080554.html)

- [九卷读书：《带人的技术》](https://www.cnblogs.com/jiujuan/p/10891893.html)
- [九卷读书：成为技术领导者 读书笔记 1](https://www.cnblogs.com/jiujuan/p/10269389.html)

- [技术管理：技术管理者的多维度能力及成长路径](https://www.cnblogs.com/jiujuan/p/11222310.html)
- [技术管理：项目开发中的几种风险管理](https://www.cnblogs.com/jiujuan/p/11493328.html)
- [技术管理：团队建设](https://www.cnblogs.com/jiujuan/p/11316993.html)
- [技术管理：项目管理概要](https://www.cnblogs.com/jiujuan/p/10597880.html)
- [好的团队和差的团队（好的产品经理和差的产品经理）](https://www.cnblogs.com/jiujuan/p/10053961.html)


### 产品
- [用价值链分析法: 分析软件开发的价值定位、赋能及创业杂感](https://www.cnblogs.com/jiujuan/p/13676593.html)
- [九卷读书：《产品设计思维：电商产品设计全攻略》读书笔记](https://www.cnblogs.com/jiujuan/p/14452748.html)
- [九卷读书：《跨越鸿沟》-产品和技术生命周期一点思考](https://www.cnblogs.com/jiujuan/p/12884881.html)
- [九卷读书[产品]：产品的视角-产品思维框架](https://www.cnblogs.com/jiujuan/p/11663782.html)
- [九卷读书[产品]: 产品的视角-产品经理能力模型](https://www.cnblogs.com/jiujuan/p/11627990.html)
- [[产品]：腾讯8分钟产品课](https://www.cnblogs.com/jiujuan/p/12173264.html)

- [需求一直做不完，怎么办？](https://www.cnblogs.com/jiujuan/p/11953572.html)

## 学习成长
- [资深技术Leader曹乐：如何成为技术大牛](https://www.cnblogs.com/jiujuan/p/12036029.html)
- [“我”这个程序员天天curd，怎么才能成长，职业生涯怎么发展？](https://www.cnblogs.com/jiujuan/p/13879947.html)
- [《阿里感悟》- 技术人员的职业规划](https://www.cnblogs.com/jiujuan/p/9278742.html)

## java
- [java基础学习：java中的反射](https://www.cnblogs.com/jiujuan/p/16659488.html)
- [SpringBoot 配置文件使用详解](https://www.cnblogs.com/jiujuan/p/16700983.html)
- [SpringBoot学习-图文并茂写Hello World](https://www.cnblogs.com/jiujuan/p/11654567.html)

## 协议
- [HTTP2 协议长文详解](https://www.cnblogs.com/jiujuan/p/16939688.html)
- [WebSocket 协议详解](https://www.cnblogs.com/jiujuan/p/16174566.html)
- [TCP协议的流量控制和拥塞控制](https://www.cnblogs.com/jiujuan/p/12210218.html)
- [TCP/IP的确认号,序列号和超时重传的学习笔记](https://www.cnblogs.com/jiujuan/p/12203836.html)
- [(传输层)tcp协议](https://www.cnblogs.com/jiujuan/p/8988645.html)
- [http协议简介](https://www.cnblogs.com/jiujuan/p/8867476.html)

- [Linux网络协议栈(一)——Socket入门(1)](https://www.cnblogs.com/jiujuan/p/9388515.html)
- [Linux网络协议栈(一)——Socket入门(2)](https://www.cnblogs.com/jiujuan/p/9388517.html)

## 九卷读书

 - [九卷读书：《复盘-对过去的事情做思维演练》读书笔记](https://www.cnblogs.com/jiujuan/p/13209745.html)
 - [九卷读书：淘宝从小到大的发展 -重读《淘宝技术这十年》](https://www.cnblogs.com/jiujuan/p/13025386.html)
 - [九卷读书：金字塔原理读书笔记 一](https://www.cnblogs.com/jiujuan/p/12967359.html)
 - [九卷读书：你的灯亮着吗? 发现问题的真正所在](https://www.cnblogs.com/jiujuan/p/12092391.html)
 - [九卷读书:《高效能人士的7个习惯》脑图](https://www.cnblogs.com/jiujuan/p/10949532.html)
 - [九卷读书：商业模式画布](https://www.cnblogs.com/jiujuan/p/10982384.html)

