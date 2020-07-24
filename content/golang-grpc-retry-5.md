**Golang gRPC学习(05): retry重试**



## 什么是重试
如果服务出现了错误，主要是网络，服务器出现了短暂异常的时候，该怎么办？
我们都会人工或者自动的重新连接服务试试，看服务是否恢复可用了。

这种重新进行连接服务的一种方式就是重试。如果是在微服务里，应该属于微服务治理的范畴。
重试是处理网络服务出现暂时不可用的一种方法。

## 怎么进行重试
**第一**：你要能知道网络出现了错误。
怎么才能知道网络出现了错误呢？
一种是你主动探测网络是否可用，比如说健康检查；一种是根据定义的错误码来进行重试。比如http的一些错误码。

**第二**：根据上面的探测或者错误码决定是否重试。
因为不是所有的错误都要进行重试，我们要根据具体情况来做决定是否重试。

**第三**：重试的策略
就是怎么进行重试。
在gRPC的这份[设计](https://github.com/grpc/proposal/blob/master/A6-client-retries.md#detailed-design)中，主要有2中重试策略：
1. Retry Policy，出错时立即重试

2. Hedging Policy，定时发送并发的多个请求，根据请求的响应情况决定是否发送下一个同样的请求，还是返回（好像该策略目前未实现）

3. 在Retry和Heading基础上的限流

客户端的重试策略和限流策略都是用一个配置文件配置 - service config，这个配置文件用proto定义了一些字段和格式，文件在 [grpc/service_config/service_config.proto](https://github.com/grpc/grpcproto/blob/master/grpc/service_config/service_config.proto) 中，它最终会解析为一个json格式，[proto->json 规则文档](https://developers.google.com/protocol-buffers/docs/proto3#json)。

一些Service Config文档例子：[doc/service_config](https://github.com/grpc/grpc/blob/master/doc/service_config.md)

## 例子
参考官方的例子把我们前面的hello word程序用retry策略改写下。

#### 客户端
主要是要在 `Dial()` 建立连接的这个函数里加一个参数 `grpc.WithDefaultServiceConfig()` ，这个就是配置重试策略的参数。
```golang
conn, err := grpc.Dial(Address, grpc.WithInsecure(),grpc.WithDefaultServiceConfig(retryPolicy))
```
里面的参数 `retryPolicy` 是一个json的字符串，这个就是service config：
```json
retryPolicy = `{
        "methodConfig":[{
           "name":[{"service": "grpc-tutorial.05retry.hello.hello"}],
           "waitForReady": true,
           "retryPolicy": {
                "MaxAttempts": 4,
                "InitialBakckoff": ".01s",
                "MaxBackoff": ".01s",
                "BackoffMultiplier": 1.0,
                 "RetryableStatusCodes": ["UNAVAILABLE"]
            }
    }]}`
```
- MaxAttempts：
最多请求次数。这里设置为4，一次原始请求，三次重试请求。

>MaxAttempts必须是大于1的整数，对于大于5的值会被视为5。

- InitialBakckoff, BackoffMultiplier, MaxBackoff
这3个参数要结合看, 意思是在进行下一次重试请求前，会计算需要等待的时间，计算公式为：
  - 第一次重试间隔是 `random(0, InitialBakckoff)`
  - 第 n 次的重试间隔为 `random(0, min( InitialBakckoff*BackoffMultiplier*(n-1) , MaxBackoff ))`
  
>InitialBakckoff 和 MaxBackoff 必须指定，并且必须具有大于0。
>BackoffMultiplier 必须指定，并且大于零。

- RetryableStatusCodes
会根据这个 `RetryableStatusCodes` 来判断是否进行重试。这里设置为 `UNAVAILABLE`，会根据这个状态来进行重试。

>retryableStatusCodes 必须制定为状态码的数据，不能为空，并且没有状态码必须是有效的 gPRC 状态码，可以是整数形式，并且不区分大小写 ([14], [“UNAVAILABLE”], [“unavailable”)

#### 服务端
在服务端我要制造重试的情况出来，主要是 failRequest() 这个函数，改写一下程序：
```go
type failServer struct {
    pb.UnimplementedHelloServer
    mu sync.Mutex

    reqNum uint
    reqMax uint
}

func (f *failServer) failRequest() error {
    f.mu.Lock()
    defer f.mu.Unlock()
    f.reqNum++
    if (f.reqMax > 0) && (f.reqNum % f.reqMax == 0) {
        return nil
    }
    return status.Errorf(codes.Unavailable, "failRequest: failing it")
}
```
然后在main函数初始化一下这个failServer struct：
```go
failService := &failServer{
        reqNum: 0,
        reqMax: 4,
}
```

完整的例子在：[这里github](https://github.com/jiujuan/grpc-tutorial/tree/master/05retry)

运行看看：
先运行服务端：go run server/main.go

在运行客户端：GRPC_GO_RETRY=on go run client/main.go

注意：
> 运行客户端一定要在环境变量里加上 GRPC_GO_RETRY=on


可是报错了：
 sayhello err: rpc error: code = Unavailable desc = failRequest: failing it
exit status 1

而且服务端也只打印了一条信息：
request failed num:  1

为什么值是1，不是4？？ 好奇怪