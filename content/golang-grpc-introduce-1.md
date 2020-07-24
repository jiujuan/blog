**golang gRPC学习(01)：gRPC介绍**


## gRPC 是什么
[gRPC](https://github.com/grpc/grpc)是goole开源的一个RPC框架和库，支持多语言之间的通信。底层通信采用的是 HTTP2 协议。gRPC在设计上使用了 [ProtoBuf](https://github.com/google/protobuf) 这种接口描述语言。这种IDL语言可以定义各种服务，google还提供了一种工具 protoc 来编译这种IDL语言，生成各种各样的语言来操作服务。


## gPRC特点

- 定义服务简单，可以很快的搭建出一个RPC调度的服务
- gRPC是与语言无关，平台无关的。你定义好了一个protobuf协议，就可以用protoc生成不同语言的协议框架
- 使用HTTP2协议，支持双向流。客户端和服务端可以双向通信

![image.png](https://cdn.nlark.com/yuque/0/2019/png/127300/1567090637934-664fde50-b6fc-4f61-a5c0-d88b3362206f.png#align=left&display=inline&height=548&name=image.png&originHeight=548&originWidth=849&size=82574&status=done&width=849)


## gRPC为了解决什么问题
gRPC其实就是一种RPC，解决服务的远程过程调用问题。<br />它使得我们远程调用就像调用本地函数一样，程序员无需关心交互的细节。


## RPC与RESTful区别是什么
- 在客户端和服务端通信还有一种基于http协议的 RESTful 架构模式，RESTful一般是对于资源的操作，它是名词（资源地址），然后添加一些动作对这些资源进行操作。<br />而RPC是基于函数，它是动词。

- RPC一般基于TCP协议，当然gRPC是基于HTTP2，但它也是比HTTP协议更加有效率和更多特性。<br />RESTful一般都是基于HTTP协议

- 传输方面：自定义的TCP协议或者使用HTTP2协议，报文体积更小，所以传输效率更高<br />                RESTful一般基于http协议，报文体积大

- gRPC用的是protobuf的IDL语言，会编码为二进制协议的数据，而RESTful一般是用json的数据格式，所以json格式的编解码更耗时


## 为什么还是有很多公司用RESTful
以函数为基础的模式（RPC），如果服务端添加了一个函数，那么客户端也必须添加一个函数。
而以资源为操作基础的RESTful，URI对应的服务变化了，客户端一般是不需要关心和更新的，它对客户端是透明的。
所以RPC的操作变更，有时比RESTful的复杂，因为RCP需要2边都要添加代码，也就增加了工作量。

并且json格式是可阅读的格式，gRPC编码后是二进制格式，对人不友好，不利于阅读.
调试排错RPC也比RESTful难，毕竟HTTP大家都很熟悉，而RPC协议一般是自定义协议，需要程序员自己去研究。


## 那怎么选择呢？RPC OR RESTful
一般公司内部服务并发大，效率要求比较高的，可以用RPC。所以一般大公司内部用RPC比较多。不过大公司对外提供的api很多都是基于http的RESTful，为什么？因为这个简单、易懂，大家都熟悉，使用就方便。

而业务量比较小，并发小，可以直接用RESTful，json，不管是公司内部服务间通信，还是对外提供api服务。

**所以选择：都是根据自己公司业务需求来判断，适合使用哪种，就选择哪种，不要盲目跟风，造成公司不必要的资源浪费。**
<br />

> golang中使用RESTful，用json传输数据，一般使用 https://github.com/gorilla/mux , https://github.com/gin-gonic/gin, https://github.com/labstack/echo，https://github.com/emicklei/go-restful 等
> golang中的RPC现在一般使用g家的gRPC,  https://grpc.io/, 毕竟大家都相信g家的技术实力 - -!


## Protocol Buffers
这是google开源的并且非常成熟的用于数据结构序列化的框架。[Protocol Buffers在github上的地址](https://github.com/protocolbuffers/protobuf).
使用protocol来序列化数据，需要编写一个proto文件，在proto文件中定义你的服务，rpc操作，返回和请求数据格式。
protocol buffers的数据是一个结构化的消息，每个消息都是一小的逻辑信息的记录，消息中包含了一系列的键值对，称之为属性，一个简单的例子：

```
message Person {
   string name = 1;
   int32 age = 2;
   bool isPay = 3;       
}
```

定义了你传输的数据结构，就可以用`protoc`来生成对应语言的代码。这些代码提供了方法，可以操作每个属性，可以将数据序列化为元数据传输给对方，也可以将对方发送过来的元数据进行解析。
例子：
```
// The greeter service definition.
service Greeter {
   //Sends a greeting
   rpc SayHello (HelloRequest) returns (HelloReply) {}
}
 
// The request message containing the user's name.
message HelloRequest {
   string name = 1;
}
 
//The response message containing the greetings
message HelloReply {
   string message = 1;
}

```
gRPC可以使用带插件的 protoc 命令，将你编写的proto文件生成代码。使用gRPC的插件的时候，你可以生成gRPC的客户端和服务端代码，和一般的protocol buffers代码一样，
可以构建，序列化，反序列化消息。

Protocol Buffer 官方文档： https://developers.google.com/protocol-buffers/docs/overview
从这里也可以获取 protoc 插件，从而生成你擅长语言的代码。

## gRPC的四种定义方式

gRPC允许用户定义四种形式的rpc方法(原文参照：https://grpc.io/docs/guides/concepts.html)：

A.客户端发送请求到服务端，然后服务端给出一个响应，就像一个普通的方法定义一样，如下所示：

```
rpc SayHello(HelloRequest) returns (HelloResponse) {};
```
B.服务端的流式rpc：客户端发送一个请求到服务端，然后得到一个流用于读取服务端的的消息，客户端从返回的流中读取所有的信息，如下所示：

```
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse) {};
```

C.客户端流式rpc： 客户端使用流将信息发送个服务端，只要客户端发送完所有的信息给服务器，就开始等待服务端的响应，如下所示：

```
rpc LotsOfGreeting(stream HelloRequest) returns (HttpResponse) ;
```

D.双向流式rpc：服务端与客户端都是用读写流发送数据给对方。这两个流式相互独立的，所以他们的读写可以是任意顺序的，例如：服务端在接受到客户的所有的信息之前就已经开始响应，
也可以先读取数据然后再写数据，或者其他任何组合，如下所示：

```
rpc SayHello(stream HelloRequest) returns (HelloResponse) ;
```

