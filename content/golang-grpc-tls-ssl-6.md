Golang gRPC学习(06): TLS/SSL认证

## 先制作证书
#### 制作私钥 (.key)

**制作server.key**
\#  openssl ecparam -genkey -name secp384r1 -out server.key

#### 自签名公钥(x509)
\# openssl req -new -x509 -sha256 -key server.key -out server.pem -days 3650
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a 
DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.

**自定义信息：**
\-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:gprc-auth-name
Email Address []:

## 定义proto
就用前面的hello的proto文件，简化下：
```go
syntax = "proto3";

package hello;

//定义hello服务
service hello {
    //定义SayHello方法
    rpc SayHello(HelloRequest) returns (HelloResponse){}
}

//HelloRequest 请求结构体
message HelloRequest {
    string name = 1;
}

//HelloResponse 响应结构体
message HelloResponse {
    string message = 1;
}
```
生成go文件：
`protoc -I . --go_out=plugins=grpc:. ./hello.proto`


## 编写代码

>完整示例代码在 [github上](
https://github.com/jiujuan/grpc-tutorial/tree/master/06TLSauth)

### 服务端代码
加上TLS，主要是这行：
```go
creds, err := credentials.NewServerTLSFromFile("../keys/server.pem", "../keys/server.key")
```

### 客户端代码
关键代码：
```go
creds, err := credentials.NewClientTLSFromFile("../keys/server.pem", "gprc-auth-name")
```
**注意**：

这里第二个参数是你上面生成`自签名公钥(x509)`填写的：Common Name (e.g. server FQDN or YOUR name) []: `gprc-auth-name`， 这个 `grpc-auth-name`，并且填写稍微复杂些。

我生成公钥时填写简单的单词，然后运行时就报错了，错误信息如下：
> rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: authentication handshake failed: x509: Common Name is not a valid hostname: grpc name"

连接时加上 `grpc.WithTransportCredentials` 参数：
```go
conn, err := grpc.Dial(Address, grpc.WithTransportCredentials(creds))
```

## 运行
1.进入到 server 目录， 运行命令：
`go run main.go`

2.进入到 client 目录，运行命令：
`go run main.go`

output：
Hello  World, auth!

## 参考
- [Golang gRPC实践 连载四 gRPC认证](https://segmentfault.com/a/1190000007933303)
