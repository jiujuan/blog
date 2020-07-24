Golang gRPC学习(07): interceptor拦截器

## 什么是拦截器
顾名思义肯定要拦截一些信息。其实这个跟java的Servlet Filter 有点像，就是对信息进行预处理：在处理信息前进行预处理，或者处理信息后在进行处理。
其实这种设计思想用的非常普遍，spring里面也有，前置处理后置处理。

gPRC里面的拦截器，也是同理。执行一段代码，在这段代码的前面或者后面，执行另外一段代码。其实在这篇文章[go middleware中间件](https://www.cnblogs.com/jiujuan/p/13069995.html) 也有类似的实现，不过它是从另外一个角度来说为什么要实现中间件，是一个http middleware的实现，可以去看看。

**但是思想原理是相同的**。

主要实现方法：go的闭包

## 用来做啥
gRPC中的拦截器可以用来做日志，认证啊，重试，metric等等功能。

## gRPC拦截器介绍

> **grpc version: 1.23.0**

### gRPC里有2种拦截器
- **unary interceptor**：unary 拦截器，拦截普通rpc调用
- **stream interceptor**：stream 拦截器，拦截stream rpc调用

但是它又分为客户端和服务端。

在 [https://godoc.org/google.golang.org/grpc](godoc.org) 上就可以看到它的所有拦截器方法。

- func StreamInterceptor(i StreamServerInterceptor) ServerOption
- func UnaryInterceptor(i UnaryServerInterceptor) ServerOption

### 客户端主要拦截器定义
- [type StreamClientInterceptor](https://github.com/grpc/grpc-go/blob/master/interceptor.go#L39)
```go
type StreamClientInterceptor func(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, streamer Streamer, opts ...CallOption) (ClientStream, error)
```

- [type UnaryClientInterceptor](https://github.com/grpc/grpc-go/blob/master/interceptor.go#L31)
```go
type UnaryClientInterceptor func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, invoker UnaryInvoker, opts ...CallOption) error
```

### 服务端主要拦截器定义
- [type UnaryServerInterceptor](https://github.com/grpc/grpc-go/blob/master/interceptor.go#L68)
```go
type UnaryServerInterceptor func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (resp interface{}, err error)
```

- [type StreamServerInterceptor](https://github.com/grpc/grpc-go/blob/master/interceptor.go#L77)
```go
type StreamServerInterceptor func(srv interface{}, ss ServerStream, info *StreamServerInfo, handler StreamHandler) error
```

### 使用
创建连接时，使用 [WithUnaryInterceptor](https://github.com/grpc/grpc-go/blob/master/dialoptions.go#L436) 和 [WithStreamInterceptor](https://github.com/grpc/grpc-go/blob/master/dialoptions.go#L455) 来设置interceptor。

**例子**：
```go
conn, err := grpc.Dial(
    grpc.WithUnaryInterceptor(unaryInterepotr),
    grpc.WithStreamInterceptor(streamInterceptor)
)
```

上面的只能设置一个interceptor，如果有多个拦截器怎么办？用**链式拦截器**设置。

**链式拦截器**：
[WithChainUnaryInterceptor](https://github.com/grpc/grpc-go/blob/master/dialoptions.go#L447) 和  [WithChainStreamInterceptor](https://github.com/grpc/grpc-go/blob/master/dialoptions.go#L466)

例子：
```go
conn, err := grpc.Dial(
    grpc.WithChainUnaryInterceptor(unaryInterepotrOne, unaryIntereptorTwo),
    grpc.WithChainStreamInterceptor(streamInterceptorOne, streamInterceptorTwo)
)
```
第一个interceptor为最外层，最后一个为最内层。

上面都是在客户端加上拦截器，服务端加在什么地方？加在 `grpc.NewServer` 里：
```go
grpc.NewServer(grpc.UnaryInterceptor(unaryInterceptor), grpc.StreamInterceptor(streamInterceptor))
```
### 例子：
完整代码例子在 [这里github](https://github.com/jiujuan/grpc-tutorial/tree/master/07interceptor)

**例子1：**
写一个简单的例子，还是用前面的hello.proto定义服务。

客户端增加一个拦截函数：
```go
// clientInterceptor，修改客户端发送给服务端的内容， 然后把服务端发送给客户端的内容也修改小
func clientInterceptor(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn,
    invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
    // 修改客户端发送给服务端的内容
    if re, ok := req.(*pb.HelloRequest);ok {
        name := strings.Replace(re.Name, "World", "big big world",1)
        req = &pb.HelloRequest{Name: name}
    }

    err := invoker(ctx, method, req, reply, cc, opts...)
    if err != nil {
        log.Println("invoke error: ", method, err)
        return err
    }

    // 修改服务端发送给客户端的内容
    if replyMsg,ok:=reply.(*pb.HelloResponse);ok {
        msg := strings.Replace(replyMsg.Message, "words", "english words", 1)
        replyMsg.Message = msg
    }
    return nil
}
```

然后在main函数的Dial里加上拦截函数：
```go
conn, err := grpc.Dial(Address, grpc.WithInsecure(), grpc.WithUnaryInterceptor(clientInterceptor))
```
服务端方法没变，只是增加了一些打印的信息和返回的信息。


#### 运行：
-  **没有**加上拦截函数时：

服务端：
>go run ./server/main.go
>Listen on   :50001
>server Req name: Server send words: Hi, World

客户端：
>go run client/main.go
>client Send name:  Hi, World
>client Recv name:  Server send words: Hi, World

-  **加上**拦截函数时：

服务端：
>go run ./server/main.go
>Listen on   :50001
>server Req name: Server send words: Hi, big big world

客户端：
>go run client/main.go
>client Send name:  Hi, World
>client Recv name:  Server send english words: Hi, big big world

加上拦截函数后，发送和接收的信息都改变了。说明我们的拦截函数起作用了。

**例子2：**

在服务端也加上拦截函数：
在 server/main.go 里加上如下函数：

```go
func UnaryServerInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo,
             handler grpc.UnaryHandler) (interface{}, error) {
    log.Println("before handling. info: ", info)
    resp, err := handler(ctx, req)
    log.Println("after handling. resp: ", resp)
    return resp, err
}

func StreamServerInterceptor(srv interface{}, ss grpc.ServerStream, info *grpc.StreamServerInfo,
    handler grpc.StreamHandler) error {
     log.Println("before handling. Info: ", info)
     err := handler(srv, ss)
     log.Println("after handling. err: ", err)
     return err
}
```

main函数里添加：
```go
grpc.NewServer(grpc.StreamInterceptor(StreamServerInterceptor), 
grpc.UnaryInterceptor(UnaryServerInterceptor))
```
客户端代码不变。

自己可以运行看看输出结果是什么。

更多的操作例子请看[这里](https://github.com/grpc/grpc-go/tree/master/examples/features/interceptor)

这里还有一个比较老的中间件集合，https://github.com/mercari/go-grpc-interceptor，有兴趣的可以去看看。

> 完整示例代码例子在 [这里github](https://github.com/jiujuan/grpc-tutorial/tree/master/07interceptor)
## 参考
- [grpc-go interceptor](https://github.com/grpc/grpc-go/tree/master/examples/features/interceptor)
- [pkg.go.dev grpc](https://pkg.go.dev/google.golang.org/grpc?tab=doc)
- [godoc grpc](https://godoc.org/google.golang.org/grpc)
- [write grpc interceptro in go](https://medium.com/@shijuvar/writing-grpc-interceptors-in-go-bf3e7671fe48)