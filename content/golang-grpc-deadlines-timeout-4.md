**Golang gRPC学习(04): Deadlines超时限制**

## 为什么要使用Deadlines
当我们使用gRPC时，gRPC库关系的是连接，序列化，反序列化和超时执行。Deadlines 允许gRPC客户端设置自己等待多长时间来完成rpc操作，直到出现这个错误 `DEADLINE_EXCEEDED`。但是在正常情况下，这个DEADLINE_EXCEEDED默认设置是一个很大的数值。
一些语言的API用deadline，一些用 timeout。

在正常情况下，你没有设置deadline，那么所有的请求可能在最大请求时间过后才超时。这样你对于你的服务器资源，可能存在风险，比如内存，可能因为这个长期运行的服务而增长很快，从而耗尽资源。
为了避免这种情况，需要给你的客户端请求程序设置一个默认的超时时间，在这段时间内请求没有返回，那么就超时报错。

## 怎么使用？Deadlines使用步骤
### 使用步骤：
**1.设置deadlines**
```go
var deadlineMs = flag.Int("deadline_ms", 20*1000, "Default deadline in milliseconds.")

clientDeadline := time.Now().Add(time.Duration(*deadlineMs) * time.Millisecond)
ctx, cancel := context.WithDeadline(ctx, clientDeadline)
```
**2.检查deadlines**
```go
if ctx.Err() == context.Canceled {
        return status.New(codes.Canceled, "Client cancelled, abandoning.")
}
```

### 具体使用：
**1. 建立连接时超时控制：**
客户端建立连接时，使用的Dial()函数，它位于
google.golang.org/grpc/clientconn.go 中，我们看看这个函数内容：

```go
func Dial(target string, opts ...DialOption) (*ClientConn, error) {    
   return DialContext(context.Background(), target, opts...)
}
```
它里面调用的 DialContext() 函数，这个函数非常长，他们在同一个文件中，它是实际执行的函数，这里面就有context的timeout和Done相关操作。你也可以到`google.golang.org/grpc/clientconn.go`文件中去看看这个函数`DialContext`具体是干嘛的。

使用的时候传入设置timeout的context，如下：
```go
ctx, cancel := context.Timeout(context.Bakcground(), time.Second*5)
defer cancel()
conn, err := grpc.DialContext(ctx, address, grpc.WithBlock(), grpc.WithInsecure())
```
>- grpc.WithInsecure() ，这个参数啥意思？
>gRPC是建立在HTTP/2上的，所以对TLS提供了很好的支持。如果在客户端建立连接过程中设置 `grpc.WithInsecure()` 就可以跳过对服务器证书的验证。写练习时可以用这个参数，但是在真实的环境中，不要这样做，因为有泄露信息的风险。
>- grpc.WithBlock()
>这个参数会阻塞等待握手成功。
>因为用Dial连接时是异步连接，连接状态为正在连接，如果设置了这个参数就是同步连接，会阻塞等待握手成功。
>这个还和超时设置有关，如果你没有设置这个参数，那么context超时控制将会失效。


**2. 调用时超时：**
函数的调用超时控制
```go
ctx, cancel := context.WithTimeout(context.TODO(), time.Second*5)
defer cancel()
result, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
```

## 实例
用grpc官方的例子来练习下

目录结构:
```
grpc-tutorial
    -- 04deadlines
        --client
            - main.go
        --server
           - main.go
       --proto/echo
           - echo.proto
           - echo.pb.go
```

#### 1.首先是定义服务 echo.proto
```bash
syntax = "proto3";
package echo;message 

EchoRequest {   
  string message = 1;
}

message EchoResponse {   
  string message = 1;
}

service Echo {    
  rpc UnaryEcho(EchoRequest) returns (EchoRequest) {}    
  rpc ServerStreamingEcho(EchoRequest) returns (stream EchoResponse) {}   
  rpc ClientStreamingEcho(stream EchoRequest) returns (EchoResponse) {}    
  rpc BidirectionalStreamingEcho(stream EchoRequest) returns (stream EchoResponse){}
}
```
进入到proto/echo目录，生成go文件，命令如下：
> protoc -I . --go_out=plugins=grpc:. ./echo.proto

#### 2.客户端
client\main.go
有2个主要的函数，2端都是stream和都不是stream，先看都不是stream的函数

**都不是stream的函数**
```go
// unaryCall 不是stream的请求
func unaryCall(c pb.EchoClient, requestID int, message string, want codes.Code) {   
    ctx, cancel := context.WithTimeout(context.Background(), time.Second) //超时设置   
    defer cancel()   

    req := &pb.EchoRequest{Message: message} //参数部分

    _, err := c.UnaryEcho(ctx, req) //调用函数发送请求给服务端
    got := status.Code(err)   //
    fmt.Printf("[%v] wanted = %v, got = %v\n", requestID, want, got)
}
```
上面的code设置在文件 grpc/codes/codes.go
```go
type Code uint32

const (
        OK Code = 0
        Canceled Code = 1
        Unknown Code = 2
        InvalidArgument Code = 3
        DeadlineExceeded Code = 4
   ... ...
)
```
**2端都是stream的函数：**
```go
// streamingCall，2端都是stream
func streamingCall(c pb.EchoClient, requestID int, message string, want codes.Code) {
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)//超时设置
    defer cancel()

    stream, err := c.BidirectionalStreamingEcho(ctx)//双向stream
    if err != nil {
        log.Printf("Send error : %v", err)
        return
    }

    err = stream.Send(&pb.EchoRequest{Message: message})//发送
    if err != nil {
        log.Printf("Send error : %v", err)
        return
    }

    _, err = stream.Recv() //接收
   
    got := status.Code(err)
    fmt.Printf("[%v] wanted = %v, got = %v\n", requestID, want, got)
}
```
**main 执行函数**
```go
func main() {
    flag.Parse()

    conn, err := grpc.Dial(*addr, grpc.WithInsecure(), grpc.WithBlock())
    if err != nil {
        log.Fatalf("did not connect : %v ", err)
    }
    defer conn.Close()

    c :=pb.NewEchoClient(conn)

    // 成功请求
    unaryCall(c, 1, "word", codes.OK)
    // 超时 deadline
    unaryCall(c, 2, "delay", codes.DeadlineExceeded)
    // A successful request with propagated deadline
    unaryCall(c, 3, "[propagate me]world", codes.OK)
    // Exceeds propagated deadline
    unaryCall(c, 4, "[propagate me][propagate me]world", codes.DeadlineExceeded)
    // Receives a response from the stream successfully.
    streamingCall(c, 5, "[propagate me]world", codes.OK)
    // Exceeds propagated deadline before receiving a response
    streamingCall(c, 6, "[propagate me][propagate me]world", codes.DeadlineExceeded)
}
```

#### 3.服务端
**定义一个struct**
```go
type server struct {    
    pb.UnimplementedEchoServer    
    client pb.EchoClient    
    cc *grpc.ClientConn
}
```

**2端不是stream的函数：**
```go
func (s *server) UnaryEcho(ctx context.Context, req *pb.EchoRequest)(*pb.EchoResponse, error) {
    message := req.Message
    if strings.HasPrefix(message, "[propagate me]") {//判断接收的值
        time.Sleep(800 * time.Millisecond)
        message := strings.TrimPrefix(message, "[propagate me]")
        return s.client.UnaryEcho(ctx, &pb.EchoRequest{Message:message}) //<1>
    }

    if message == "delay" {
        time.Sleep(1500 * time.Millisecond) // message=delay 时睡眠1500毫秒，大于client的设置的1秒，这里就超时了
    }

    return &pb.EchoResponse{Message:message}, nil
}
```
上面函数标注 <1> 这个地方比较有意思，当client端发送的字符串包含 [propagate me] 字符串时，先睡眠800毫秒，然后在重新执行客户端请求服务端的函数 s.client.UnaryEcho() , 在次运行到服务端的 UnaryEcho()，客户端已经超时了。
也就是说client/main.go 先请求了一次服务端，然后在server/main.go 的函数 `func (s *server) UnaryEcho(ctx context.Context, req *pb.EchoRequest) `又执行了一次请求服务端，所以会导致超时。

**2端设置stream的函数：**
```go
func (s *server) BidirectionalStreamingEcho(stream pb.Echo_BidirectionalStreamingEchoServer) error {
    for {
        req, err := stream.Recv()
        if err == io.EOF {
            return status.Error(codes.InvalidArgument, "request message not received")
        }
        if err != nil {
            return err
        }

        message := req.Message
        if strings.HasPrefix(message, "[propagate me]") {
            time.Sleep(800 * time.Millisecond)
            message = strings.TrimPrefix(message, "[propagate me]")
            res, err := s.client.UnaryEcho(stream.Context(), &pb.EchoRequest{Message:message})//再次执行客户端请求服务端函数，这里可能会超时
            if err != nil {
                return err
            }
            stream.Send(res)
        }

        if message == "delay" {
            time.Sleep(1500 * time.Millisecond)
        }
        stream.Send(&pb.EchoResponse{Message:message})
    }
}
```
main函数
```go
func main() {
    flag.Parse()

    address := fmt.Sprintf(":%v", *port)
    lis, err := net.Listen("tcp", address)
    if err != nil {
        log.Fatalf("failed to listen: %v ", err)
    }

    echoServer := newEchoServer()
    defer echoServer.Close()

    grpcServer := grpc.NewServer()
    pb.RegisterEchoServer(grpcServer, echoServer)

    if err := grpcServer.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v ", err)
    }
}
```

#### 执行
先运行 /server/main.go , go run main.go

在运行 /client/main.go, go run main.go

执行结果：
```
 go run main.go
[1] wanted = OK, got = OK
[2] wanted = DeadlineExceeded, got = DeadlineExceeded
[3] wanted = OK, got = Unavailable
[4] wanted = DeadlineExceeded, got = Unavailable
[5] wanted = OK, got = Unavailable
[6] wanted = DeadlineExceeded, got = Unavailable
```

## 参考：
- [gRPC and Deadlines](https://grpc.io/blog/deadlines/)
- [grpc-go deadline](https://github.com/grpc/grpc-go/tree/master/examples/features/deadline)