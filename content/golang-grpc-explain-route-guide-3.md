**Golang gRPC学习(03): grpc官方示例程序route_guide简析**

代码主要来源于grpc的官方examples代码：
[route_guide](https://github.com/grpc/grpc-go/tree/master/examples/route_guide)
https://github.com/grpc/grpc-go/tree/master/examples/route_guide

## 服务定义 RouteGuide 

```go
service RouteGuide {
  // A simple RPC.
  //
  // Obtains the feature at a given position.
  //
  // A feature with an empty name is returned if there's no feature at the given
  // position.
  rpc GetFeature(Point) returns (Feature) {}
  
  // A server-to-client streaming RPC.
  //
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  
  // A client-to-server streaming RPC.
  //
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
  
  // A Bidirectional streaming RPC.
  //
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
}
```

**从定义里看：**

rpc GetFeature(Point) returns (Feature) ： 定义最简单的RPC服务

rpc ListFeatures(Rectangle) returns (stream Feature)： 带有 stream 的RPC服务，返回是stream

rpc RecordRoute(stream Point) returns (RouteSummary)：带有 stream 的RPC服务，客户端是stream

rpc RouteChat(stream RouteNote) returns (stream RouteNote)：2端都是stream的RPC服务

**rpc服务其他参数定义：**

```go
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}

message Rectangle {
  // One corner of the rectangle.
  Point lo = 1;
  // The other corner of the rectangle.
  Point hi = 2;
}

message Feature {
  // The name of the feature.
  string name = 1;
  // The point where the feature is detected.
  Point location = 2;
}

message RouteNote {
  // The location from which the message is sent.
  Point location = 1;
  // The message to be sent.
  string message = 2;
}

message RouteSummary {
  // The number of points received.
  int32 point_count = 1;
  // The number of known features passed while traversing the route.
  int32 feature_count = 2;
  // The distance covered in metres.
  int32 distance = 3;
  // The duration of the traversal in seconds.
  int32 elapsed_time = 4;
}
```

## route_guide.pb.go
route_guide.pb.go，这个文件是干嘛的？
这个是grpc自动生成的文件，里面有序列化，反序列化，执行是函数。

先看看里面的接口，其中里面有2个主要接口：
- 1.RouteGuideClient interface
- 2.RouteGuideServer interface

**1.RouteGuideClient interface定义**
```go
type RouteGuideClient interface {
    GetFeature(ctx context.Context, in *Point, opts ...grpc.CallOption) (*Feature, error)

    ListFeatures(ctx context.Context, in *Rectangle, opts ...grpc.CallOption) (RouteGuide_ListFeaturesClient, error)

    RecordRoute(ctx context.Context, opts ...grpc.CallOption) (RouteGuide_RecordRouteClient, error)

    RouteChat(ctx context.Context, opts ...grpc.CallOption) (RouteGuide_RouteChatClient, error)
}
```
仔细看看，这里面定义的一些方法，都是route_guide.proto文件里的service RouteGuide里的rpc方法，而且是一一对应的。
这个就是client需要操作的方法，是grpc自动生成的。客户端请求这些方法来调用服务。

从这里可以看出，grpc把proto中定义的服务映射为了一个interface，里面包含service中定义的rpc方法。

**2.RouteGuideServer interface**
```go
type RouteGuideServer interface {
    GetFeature(context.Context, *Point) (*Feature, error)

    ListFeatures(*Rectangle, RouteGuide_ListFeaturesServer) error

    RecordRoute(RouteGuide_RecordRouteServer) error
 
    RouteChat(RouteGuide_RouteChatServer) error
}
```
这个也是与route_guide.proto文件里的service RouteGuide里的rpc方法是一一对应的。
同理，这里跟上面的RouteGuideClient一样，把service映射成了interface。

## 客户端请求client.go

看看3个用stream发送数据的函数，客户端用stream，服务端用stream，2端都用stream，

### ListFeatures(Rectangle) returns (stream Feature) 方法
我们先看看 rpc ListFeatures(Rectangle) returns (stream Feature) {} 这个rpc方法，它返回的是一个 stream，在 `client.go` 文件里它是用 `printFeatures()` 这个函数来表示的，

```go
// printFeatures lists all the features within the given bounding Rectangle.
func printFeatures(client pb.RouteGuideClient, rect *pb.Rectangle) {
    log.Printf("Looking for features within %v", rect)
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    //这里调用 ListFeatures(), rpc定义的方法，返回一个stream
    stream, err := client.ListFeatures(ctx, rect) 
    if err != nil {
        log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
    }
    
    for {//for循环不断的接收数据
        feature, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
        }
        log.Println(feature)
    }
}
```

### RecordRoute(stream Point) returns (RouteSummary)
这个 rpc RecordRoute(stream Point) returns (RouteSummary) 方法，通过stream发送消息，在 `client.go` 文件里是 `runRecordRoute()` 方法

```go
func runRecordRoute(client pb.RouteGuideClient) {
    // Create a random number of random points
    r := rand.New(rand.NewSource(time.Now().UnixNano()))
    pointCount := int(r.Int31n(100)) + 2 // Traverse at least two points
    var points []*pb.Point
    for i := 0; i < pointCount; i++ {
        points = append(points, randomPoint(r))
    }
    log.Printf("Traversing %d points.", len(points))
    
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    // 调用RecordRoute() 方法
    stream, err := client.RecordRoute(ctx)
    if err != nil {
        log.Fatalf("%v.RecordRoute(_) = _, %v", client, err)
    }
    for _, point := range points {
        // stream.Send() stream方式发送数据
        if err := stream.Send(point); err != nil {
            log.Fatalf("%v.Send(%v) = %v", stream, point, err)
        }
    }
    // 关闭
    reply, err := stream.CloseAndRecv()
    if err != nil {
        log.Fatalf("%v.CloseAndRecv() got error %v, want %v", stream, err, nil)
    }
    log.Printf("Route summary: %v", reply)
}
```

### RouteChat(stream RouteNote) returns (stream RouteNote)
这个是 rpc RouteChat(stream RouteNote) returns (stream RouteNote)，2端都是操作stream，在 `client.go` 里面 `runRouteChat()`
```go
func runRouteChat(client pb.RouteGuideClient) {
    notes := []*pb.RouteNote{
        {Location: &pb.Point{Latitude: 0, Longitude: 1}, Message: "First message"},
        {Location: &pb.Point{Latitude: 0, Longitude: 2}, Message: "Second message"},
        {Location: &pb.Point{Latitude: 0, Longitude: 3}, Message: "Third message"},
        {Location: &pb.Point{Latitude: 0, Longitude: 1}, Message: "Fourth message"},
        {Location: &pb.Point{Latitude: 0, Longitude: 2}, Message: "Fifth message"},
        {Location: &pb.Point{Latitude: 0, Longitude: 3}, Message: "Sixth message"},
    }
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    stream, err := client.RouteChat(ctx)
    if err != nil {
        log.Fatalf("%v.RouteChat(_) = _, %v", client, err)
    }
    waitc := make(chan struct{})
    go func() { //开一个协程来执行接收的动作
        for {
            in, err := stream.Recv() //stream方式接收
            if err == io.EOF {
                // read done.
                close(waitc) //读取完成close掉chan，给外面的waitc发送一个结束的信号表示协程工作已完成
                return
            }
            if err != nil {
                log.Fatalf("Failed to receive a note : %v", err)
            }
            log.Printf("Got message %s at point(%d, %d)", in.Message, in.Location.Latitude, in.Location.Longitude)
        }
    }()
    
    // 在main协程里面 strem 发送数据
    for _, note := range notes {
        if err := stream.Send(note); err != nil {
            log.Fatalf("Failed to send a note: %v", err)
        }
    }
    
    stream.CloseSend() // 关闭stream
    <-waitc //stream 接收完成通知退出协程，main协程也结束运行
}
```

## 服务端server.go
定义了一个struct，routeGuideServer struct，然后是操作这个struct，
```go
type routeGuideServer struct {
    pb.UnimplementedRouteGuideServer
    savedFeatures []*pb.Feature // read-only after initialized
    mu         sync.Mutex // protects routeNotes
    routeNotes map[string][]*pb.RouteNote
}
```

### rpc GetFeature()
这个rpc方法，客户端和服务端都不是stream方式发送，获取，
```go
func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
    for _, feature := range s.savedFeatures {
        if proto.Equal(feature.Location, point) {
            return feature, nil
        }
    }
    // No feature was found, return an unnamed feature
    return &pb.Feature{Location: point}, nil
}
```
里面有一个比较的函数 proto.Equal(a, b Message) bool，在 `protobuf/proto/equal.go` 里，比较2值是否相等。

### rpc ListFeatures()
rpc ListFeatures(Rectangle) returns (stream Feature)

这个rpc方法返回是一个stream，也就是服务端发送是stream方式发送，
```go
func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
    for _, feature := range s.savedFeatures {
        if inRange(feature.Location, rect) {
           //stream 方式发送
            if err := stream.Send(feature); err != nil {
                return err
            }
        }
    }
    return nil
}
```

### rpc RecordRoute
rpc RecordRoute(stream Point) returns (RouteSummary)

客户端发送（请求服务端）的数据是一个stream，那服务端server接收肯定也要用stream，

```go
func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
    var pointCount, featureCount, distance int32
    var lastPoint *pb.Point
    startTime := time.Now()
    for {
        point, err := stream.Recv() //接收stream
        if err == io.EOF {
            endTime := time.Now()
            return stream.SendAndClose(&pb.RouteSummary{
                PointCount:   pointCount,
                FeatureCount: featureCount,
                Distance:     distance,
                ElapsedTime:  int32(endTime.Sub(startTime).Seconds()),
            })
        }
        if err != nil {
            return err
        }
        pointCount++
        for _, feature := range s.savedFeatures {
            if proto.Equal(feature.Location, point) {
                featureCount++
            }
        }
        if lastPoint != nil {
            distance += calcDistance(lastPoint, point)
        }
        lastPoint = point
    }
}
```

RecordRoute 函数的参数 `pb.RouteGuide_RecordRouteServer` 是什么？
它在 route_guide.pg.go 定义的是一个 interface，里面有SendAndClose(), Recv() 方法，

```go
type RouteGuide_RecordRouteServer interface {
    SendAndClose(*RouteSummary) error
    Recv() (*Point, error)
    grpc.ServerStream
}
```

### rpc RouteChat
rpc RouteChat(stream RouteNote) returns (stream RouteNote)

这个rpc方法，客户端和服务端都是stream发送，接收，
```go
func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
    for {
        in, err := stream.Recv() // stream接收
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }
        key := serialize(in.Location)
        s.mu.Lock()
        s.routeNotes[key] = append(s.routeNotes[key], in)
        // Note: this copy prevents blocking other clients while serving this one.
        // We don't need to do a deep copy, because elements in the slice are
        // insert-only and never modified.
        rn := make([]*pb.RouteNote, len(s.routeNotes[key]))
        copy(rn, s.routeNotes[key])
        s.mu.Unlock()
        for _, note := range rn {
            if err := stream.Send(note); err != nil { //stream发送
                return err
            }
        }
    }
}
```

### newServer
```go
// 运行服务端函数，返回一个struct，这个struct又实现了interface，
//所以main函数里可以直接调用 pb.RegisterRouteGuideServer(grpcServer, newServer())
func newServer() *routeGuideServer {
    s := &routeGuideServer{routeNotes: make(map[string][]*pb.RouteNote)}
    s.loadFeatures(*jsonDBFile)
    return s
}

func main() {
    flag.Parse()
    lis, err := net.Listen("tcp", fmt.Sprintf("localhost:%d", *port))
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    var opts []grpc.ServerOption
    if *tls {
        if *certFile == "" {
            *certFile = testdata.Path("server1.pem")
        }
        if *keyFile == "" {
            *keyFile = testdata.Path("server1.key")
        }
        creds, err := credentials.NewServerTLSFromFile(*certFile, *keyFile)
        if err != nil {
            log.Fatalf("Failed to generate credentials %v", err)
        }
        opts = []grpc.ServerOption{grpc.Creds(creds)}
    }
    grpcServer := grpc.NewServer(opts...)
    pb.RegisterRouteGuideServer(grpcServer, newServer())
    grpcServer.Serve(lis)
}
```

### 服务端一些辅助函数
`server.go`  文件里面还有一些辅助函数：

**1.loadFeatures**
```go
// 加载json形式的 features 数据，经纬度，名称
func (s *routeGuideServer) loadFeatures(filePath string)
```

下面这种格式的数据，跟 `route_guide.proto` 里 `message Feature` 数据定义一致
```json
{
"location": {
        "latitude": 407838351,
        "longitude": -746143763
    },
    "name": "Patriots Path, Mendham, NJ 07945, USA"
}
```

**2.toRadians**
计算弧度

3.calcDistance
计算距离

4.inRange
在范围内

5.serialize
序列化