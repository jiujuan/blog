**Golang gRPC学习(02): 编写helloworld服务**

## 安装protobuf
在windows下，直接下载release版本[https://github.com/protocolbuffers/protobuf/releases/tag/v3.9.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.9.0)

然后把bin目录加入到环境变量中

如果是在Linux下，下载对应的版本，然后编译，把编译后的文件加入env中

安装go protobuf plugin
可以参考这里： https://studygolang.com/articles/11343

```shell
go get -u github.com/golang/protobuf/protoc-gen-go
```


## 编写HelloWorld
用官方的例子： https://github.com/grpc/grpc-go/tree/master/examples/helloworld

目录结构如下：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/127300/1567099225444-ce27894e-c6e4-4033-8980-7945a39e1e47.png#align=left&display=inline&height=186&name=image.png&originHeight=186&originWidth=208&size=9236&status=done&width=208)

建立一个grpc-helloworld的目录，然后运行命令： go mod init grpc-helloworld，我们用go mod来管理包。

然后按上图建立相同的文件夹


### 编写proto文件
编写proto，名字为helloworld/hellowporld.proto 

> proto的语法：

> 参考：[https://segmentfault.com/a/1190000007917576](https://segmentfault.com/a/1190000007917576)

> 官方：[https://developers.google.com/protocol-buffers/docs/gotutorial](https://developers.google.com/protocol-buffers/docs/gotutorial)


```protobuf
syntax = "proto3";

option java_multiple_files = true;
option java_outer_classname = "HelloWorldProto";

package helloworld;

service Greet {
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloReply {
    string message = 1;
}

message HelloRequest {
    string name = 1;
}
```


### 编译proto文件
直接进入 helloworld.proto 所在的文件夹，运行命令

```shell
 protoc -I . --go_out=plugins=grpc:. ./helloworld.proto
```
然后在helloworld.proto同级目录生成了一个 helloworld.pb.go 的文件

protoc参数解释：

- -I： 指定import路径，可以指定多个-I参数，编译时按顺序查找，不指定默认当前目录
- -go_out：指定go语言的访问类
- plugins：指定依赖的插件


### 编写服务端代码

```go
package main

import (
	"context"
	pb "grpc-helloworld/helloworld"
	"log"
	"net"

	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
)

const (
	port = ":8080"
)

type server struct{}

func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Message: "Hello" + in.Name}, nil
}

func main() {
	//指定执行程序监听的端口
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	//建立 gPRC 服务器，并注册服务
	s := grpc.NewServer()
	pb.RegisterGreetServer(s, &server{})
	
        log.Println("Server run ...")
	//启动服务
	if err := s.Serve(lis); err != nil {
		log.Fatalf("fail to serve: %v", err)
	}
}
```


### 编写客户端代码

```go
package main

import (
	"context"
	pb "grpc-helloworld/helloworld"
	"log"
	"os"
    "time"

	"google.golang.org/grpc"
)

const (
	address     = "localhost:8080"
	defaultName = "world"
)

func main() {
	//连接到gRPC服务端
	conn, err := grpc.Dial(address, grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()

	//建立客户端
	c := pb.NewGreetClient(conn)

	name := defaultName
	if len(os.Args) > 1 {
		name = os.Args[1]
	}
        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	// 调用方法
	r, err := c.SayHello(context.Background(), &pb.HelloRequest{Name: name})
	if err != nil {
		log.Fatalf("couldn not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.Message)
}
```


### 测试运行

1、 打开一个shell，进入 greeter_server 目录，运行  go run main.go
```shell
$ go run main.go
2019/07/14 01:35:36 Server run ...

```

2、在开另外一个shell，进入greeter_client 目录，运行 go run main.go
```shell
$ go run main.go
2019/07/14 01:36:10 Greeting: Helloworld
```
     <br />输出了 Helloworld 说明程序运行成功了