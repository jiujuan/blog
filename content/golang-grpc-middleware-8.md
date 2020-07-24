Golang gRPC学习(08): go-grpc-middleware

## 简介
前面 第 07 节，我们简单介绍了grpc自身拦截器的使用方法。
但是互联网人的智慧是无穷的，经常想着怎么创造更好的工具来使我们的工作更简单。

如是他们创造了一个项目：
[go-grpc-middleware](https://github.com/grpc-ecosystem/go-grpc-middleware)，一个中间件集合的项目，把常用的中间件都集合起来，想用哪个可以直接调用。

**官方介绍**：
Go gRPC Middleware：gRPC Go Middleware: interceptors, helpers, utilities.

**Middleware**:

gRPC Go recently acquired support for Interceptors, i.e. middleware that is executed either on the gRPC Server before the request is passed onto the user's application logic, or on the gRPC client either around the user call. It is a perfect way to implement common patterns: auth, logging, message, validation, retries or monitoring.
(大意：官方最近开始支持拦截器，也就是中间件模式。它不仅能在gRPC服务器端请求之前做一些用户逻辑处理的工作，而且在客户端也能预处理用户逻辑。middleware是一种很好的方式来执行一些通用的操作，比如：auth，logging，message，validation，retries or monitoring)

## 功能列表
gRPC middleware 或者 拦截器，目前提供了以下的一些功能：

**Auth**
- [grpc_auth](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/auth) - 用户验证中间件

**Logging**
- [grpc_ctxtags](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/tags) - 为context增加一个a tag map功能
- [grpc_zap](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/logging/zap) -  [zap](https://github.com/uber-go/zap) 日志的gPRC中间件
- [grpc_logrus](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/logging/logrus) -  [logrus](https://github.com/sirupsen/logrus) 日志的中间件
- [grpc_kit](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/logging/kit) -  [go-kit](https://github.com/go-kit/kit/tree/master/log) 里面的日志中间件.

**Monitoring**
- [grpc_prometheus](https://github.com/grpc-ecosystem/go-grpc-prometheus)⚡ - Prometheus 客户端服务端监控中间件
- [otgrpc](https://github.com/grpc-ecosystem/grpc-opentracing/tree/master/go/otgrpc)⚡ - [OpenTracing](http://opentracing.io/) 链路追踪
- grpc_opentracing - [OpenTracing](http://opentracing.io/) 链路追踪

**Client**
- [grpc_retry](https://github.com/grpc-ecosystem/go-grpc--middleware/blob/master/retry) - 客户端重试中间件

**Server**
- [grpc_validator](https://github.com/grpc-ecosystem/go-grpc--middleware/blob/master/validator) 
- [grpc_recovery](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/recovery) 
- [ratelimit](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/ratelimit) 

## 使用说明
它支持链式的调用，看下面的服务端调用的例子：
```go
import "github.com/grpc-ecosystem/go-grpc-middleware"

myServer := grpc.NewServer(
         grpc.StreamInterceptor(grpc_middleware.ChainStreamServer(
        grpc_ctxtags.StreamServerInterceptor(),
        grpc_opentracing.StreamServerInterceptor(),
        grpc_prometheus.StreamServerInterceptor,
        grpc_zap.StreamServerInterceptor(zapLogger),
        grpc_auth.StreamServerInterceptor(myAuthFunction),
        grpc_recovery.StreamServerInterceptor(),
    )),
    grpc.UnaryInterceptor(grpc_middleware.ChainUnaryServer(
        grpc_ctxtags.UnaryServerInterceptor(),
        grpc_opentracing.UnaryServerInterceptor(),
        grpc_prometheus.UnaryServerInterceptor,
        grpc_zap.UnaryServerInterceptor(zapLogger),
        grpc_auth.UnaryServerInterceptor(myAuthFunction),
        grpc_recovery.UnaryServerInterceptor(),
    )),
)
```
客户端调用可以看下面的例1：grpc_retry 功能

## Examples
> go-grpc-middleware v1.2.0
> grpc v1.23.0

**例1：**

比如说前面实现的retry重试功能，用这里的中间件怎么实现呢？
重点在客户端的main函数，其他没什么大的变化，添加grpc_retry的拦截器代码：
```go
conn, err := grpc.Dial(Address, grpc.WithInsecure(),
        grpc.WithUnaryInterceptor(grpc_retry.UnaryClientInterceptor(
            grpc_retry.WithCodes(codes.Canceled, codes.Unavailable, codes.NotFound),
            grpc_retry.WithMax(4),
            grpc_retry.WithPerRetryTimeout(time.Second * 1),
        )),
    )
```

> 完整例子在  [这里](
https://github.com/jiujuan/grpc-tutorial/tree/master/08middleware/retry)

---

**例2：**



在客户端我们加上 auth token 验证，为每个gRPC方法调用提供认证支持。

直接在client/main.go 里添加下面代码:
```go
type Token struct {
    Value string
}

func (token *Token) GetRequestMetadata(ctx context.Context, s ...string) (map[string]string, error) {
    // 下面map中的authorization，如果你要用grpc_auth，这个key就是固定的
    // 它在 grpc-ecosystem/go-grpc-middleware/auth/metadata.go 定义
    return map[string]string{"authorization": token.Value}, nil
}

// 是否开启TLS安全传输
func (token *Token) RequireTransportSecurity() bool {
    return IsTLS
}
```
在 grpc.Dial() 函数里加上如下代码：
```go
grpc.WithPerRPCCredentials(&token)
```

>上面完整代码在 [这里](https://github.com/jiujuan/grpc-tutorial/tree/master/08middleware/multinterceptor)

探究下这个 `WithPerRPCCredentials()` 到底是什么意思？
它在`grpc/dialoptions.go`中，
```go
func WithPerRPCCredentials(creds credentials.PerRPCCredentials) DialOption {      
    return newFuncDialOption(func(o *dialOptions {           
        o.copts.PerRPCCredentials = append(o.copts.PerRPCCredentials, creds)    
    })
}
```
继续追踪，最后我们可以在 `grpc/credentials/credentials.go` 找到这个interface，它有啥作用？
前面说了，要对每个gRPC方法进行认证，那么，就需要实现这个接口。
```go
type PerRPCCredentials interface {      
  GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error)     

  RequireTransportSecurity() bool
}
```
`Authentication struct` 实现了这2个接口

**服务端**

>完整代码在 [这里github](https://github.com/jiujuan/grpc-tutorial/tree/master/08middleware/multinterceptor)

- middleware/server/auth.go  

接收验证token信息并进行相关处理。
auth 拦截器, 采用 `:authorization` header，头部的数据形式是固定的，形如 "basic/bearer token"，可以看文件 `grpc-ecosystem/go-grpc-middleware/auth/metadata.go` 定义。

AuthInterceptor() 拦截器函数定义如下：
```go
// AuthInterceptor 拦截器, 采用 `:authorization` header，头部的数据形式是固定的，形如 "basic/bearer token"
// 可以看文件 grpc-ecosystem/go-grpc-middleware/auth/metadata.go 定义
func AuthInterceptor(ctx context.Context) (context.Context, error) {
    token, err := grpc_auth.AuthFromMD(ctx, "bearer")
    log.Println("Req token val: ", token)
    if err != nil {
        return nil, err
    }
    auth, err := decodeToken(token)
    if err != nil {
        return nil, status.Errorf(codes.Unavailable, " %v ", err)
    }
    newCtx := context.WithValue(ctx, auth.User, auth)
    log.Println(newCtx.Value(auth.User))
    return newCtx, nil
}
```

- middleware/server/tls.go  

把tls的验证也独立成一个interceptor，单独进行处理。

```go
func TLSServerInterceptor() grpc.ServerOption {
    creds, err := credentials.NewServerTLSFromFile("./keys/server.pem", "./keys/server.key")
    if err != nil {
        grpclog.Fatalf("failed to generate credentials %v", err)
    }
    return grpc.Creds(creds)
}
```
上面的grpc.Creds()函数在 `google.golang.org/grpc/server.go` 文件中，返回一个ServerOption 的interface：
```go
func Creds(c credentials.TransportCredentials) ServerOption {
	return newFuncServerOption(func(o *serverOptions) {
		o.creds = c
	})
}
```

服务端的main函数，server/manin.go 中 grpc.NewServer() 函数参数干好是ServerOption的多参数：
```go
func NewServer(opt ...ServerOption) *Server
```

- middleware/server/recovery.go

recovery interceptor，panic的时候，定义返回错误。

```go
func RecoveryInterceptor() grpc_recovery.Option {
    return grpc_recovery.WithRecoveryHandler(func(p interface{}) (err error) {
        return status.Errorf(codes.NotFound, "panic : %v", p)
    })
}
```

最后在 server/man.go 里添加：
```go
s := grpc.NewServer(servermid.TLSServerInterceptor(),
        grpc.StreamInterceptor(grpc_middleware.ChainStreamServer(
            grpc_auth.StreamServerInterceptor(servermid.AuthInterceptor),
            grpc_recovery.StreamServerInterceptor(servermid.RecoveryInterceptor()),
        )),
        grpc.UnaryInterceptor(grpc_middleware.ChainUnaryServer(
            grpc_auth.UnaryServerInterceptor(servermid.AuthInterceptor),
         )),
    )
```

然后测试运行：
进入到目录 cd 08middleware/multinterceptor里，

go run server/main.go
输出
>Listen on   :50001
>Listen on  :50001  with TLS
>Req token val:  grpc-auth-user
>{test.token grpc.auth.test}

go run client/main.go
输出
>Hello  World, auth!


> 完整示例代码在 [这里github](https://github.com/jiujuan/grpc-tutorial/tree/master/08middleware/multinterceptor)

## 参考
- [gRPC认证](https://segmentfault.com/a/1190000007933303)
- [go-grpc-middleware](https://github.com/grpc-ecosystem/go-grpc-middleware)