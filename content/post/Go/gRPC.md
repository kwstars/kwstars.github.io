---
title: "gRPC"
date: 2019-12-17T10:48:05+08:00
draft: false
tags: ["gRPC"]
categories: ["Go"]
---

gRPC是可以在任何环境中运行的现代开源高性能RPC框架。 它可以通过可插拔的支持来有效地连接数据中心内和跨数据中心的服务，以实现负载平衡，跟踪，健康检查和身份验证。 它也适用于分布式计算的最后一英里，以将设备，移动应用程序和浏览器连接到后端服务。

**简单的服务定义**

定义服务使用Protocol Buffers，它是一个功能强大的二进制序列化工具集和语言

**启动快并可扩展**

只需一行即可安装运行时和开发环境，并使用该框架每秒可扩展至数百万个RPC

**跨语言和平台**

为您提供多种语言和平台的服务，可以自动生成惯用的客户端和服务器存根


**双向<!--Bi-directional-->流和集成身份验证**
双向流传输和完全集成的可插入身份验证都基于HTTP/2的传输

![enter image description here](https://grpc.io/img/landing-2.svg)

总结为以下几点
1. 使用ProtoBuf序列化，性能比JSON，XML高
2. 支持多种语言
3. 基于HTTP/2标准设计，相对其他框架，gRPC拥有更多强大功能，双向流、头压缩、多路复用

##  一、安装gRPC
### 1.1 安装Go
 [installation instructions](https://golang.org/doc/install)

### 1.2 Protocol buffer compiler
```bash
-VERSION="3.12.4"
PB_REL="https://github.com/protocolbuffers/protobuf/releases"
curl -LO $PB_REL/download/v$VERSION/protoc-$VERSION-linux-x86_64.zip
unzip protoc-$VERSION-linux-x86_64.zip -d $GOPATH

# 将GOPATH加入到环境变量，安装Go时加过的可忽略
export PATH="$PATH:$GOPATH/bin"
```

### 1.3 Go plugin for the protocol compiler
```bash
export GO111MODULE=on  # Enable module mode
go get github.com/golang/protobuf/protoc-gen-go
```

### 1.4 测试
```bash
git clone -b v1.31.0 https://github.com/grpc/grpc-go
cd grpc-go/examples/helloworld

# 运行server和clinet会有如下显示
go run greeter_server/main.go
2020/08/06 06:59:56 Received: world

go run greeter_client/main.go 
2020/08/06 06:59:56 Greeting: Hello world
```

## 二、新增gRPC service
**gRPC可以定义四种服务方法**

一元RPC，客户端向服务器发送单个请求并获得单个响应，就像普通的函数调用一样。
```protobuf
rpc SayHello(HelloRequest) returns (HelloResponse);
```

服务器流式RPC，客户端在其中向服务器发送请求后 获得 a stream to read a sequence of messages back。 客户端从返回的流中读取，直到没有更多消息为止。 gRPC保证单个RPC调用中的消息顺序。
```protobuf
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
```

客户端流式RPC，客户端在其中编写a sequence of messages，然后将它们发送到服务器。 客户端写完消息后，它将等待服务器读取消息并返回响应。 gRPC再次保证了在单个RPC调用中的消息顺序。
```protobuf
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
```

双向流式RPC，双方都使用读写流发送a sequence of messages。 这两个流是独立运行的，因此客户端和服务器可以按照自己喜欢的顺序进行读写。类似于Webocket。
```protobuf
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
```

### 2.1 修改 Protocol Buffer 文件
```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

### 2.2 生成 gRPC 代码
在 `examples/helloworld` 运行如下代码
```bash
( cd ../../cmd/protoc-gen-go-grpc && go install . )
protoc \
  --go_out=Mgrpc/service_config/service_config.proto=/internal/proto/grpc_service_config:. \
  --go-grpc_out=Mgrpc/service_config/service_config.proto=/internal/proto/grpc_service_config:. \
  --go_opt=paths=source_relative \
  --go-grpc_opt=paths=source_relative \
  helloworld/helloworld.proto
```

执行( cd ../../cmd/protoc-gen-go-grpc && go install . )会在GOPATH目录生成protoc-gen-go-grpc文件

![image-20200806072059192](https://i.imgur.com/iw8qvyd.png)

生成后的gRPC代码

![image-20200806071807516](https://i.imgur.com/9sCELhm.png)

> 我们正在过渡到[新的Go protoc插件](https://github.com/grpc/grpc-go/pull/3453)。 在过渡完成之前，您需要在重新生成.pb.go文件之前手动安装`grpc-go/cmd/protoc-gen-go-grpc`。 要跟踪此问题的进度，请参阅[Update Go快速入门＃298](https://github.com/grpc/grpc.io/issues/298)。

### 2.3 更新并运行应用程序
**更新服务端代码**

在 `greeter_server/main.go`  中添加如下代码设置 为
```go
func (s *server) SayHelloAgain(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
        return &pb.HelloReply{Message: "Hello again " + in.GetName()}, nil
}
```

**更新客户端代码**

在 `greeter_client/main.go` 的 `main()` 中的最后面添加下面代码:
```go
r, err = c.SayHelloAgain(ctx, &pb.HelloRequest{Name: name})
if err != nil {
        log.Fatalf("could not greet: %v", err)
}
log.Printf("Greeting: %s", r.GetMessage())
```

**运行代码**

```bash
go run greeter_server/main.go

go run greeter_client/main.go Kira
2020/08/06 07:41:15 Greeting: Hello Kira
2020/08/06 07:41:15 Greeting: Hello again Kira
```

## 三、gRPC stream
一元RPC（Unary RPCs）的局限是客户端发送一个请求后，必须等待服务发回响应才能继续发送下一个请求，这样无法更好的利用带宽，传递更多的请求或响应。而gRPC支持流式的请求模式来优化解决这一个问题。

 ![image-20200806082222672](https://i.imgur.com/2FvGLTo.png)

{{< gist kwstars 2cc3749cff7b2006f75eb1f040c1be4f >}}


---
[Quick Start](https://grpc.io/docs/languages/go/quickstart/)

[micro-go-book](https://github.com/longjoy/micro-go-book/tree/master/ch7-rpc)

