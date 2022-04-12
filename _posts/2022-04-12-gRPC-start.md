---
layout: post
title:  用go开始一个gRPC的简单使用
date:   2022-04-12 10:30:00 +0800
categories: gRPC
tag: note
---

* content
{:toc}

#### 准备工作

- 安装编译 proto 文件插件
```
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

- 修改 PATH，protoc编译全局使用
```
export PATH="$PATH:$(go env GOPATH)/bin"
```

检查安装结果，查看版本：
```
protoc --version
# libprotoc 3.8.0
```

#### 文件目录结构

```
example
├── helloword     
    ├── greeter_server
    │   └── main.go
    └── helloworld
        └── helloworld.proto        
```

#### 编写一个简单的proto文件
```
// helloworld.proto
syntax = "proto3";

option go_package = "google.golang.org/grpc/examples/helloworld/helloworld";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {
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

#### 编译 proto 文件
```
# 编译成功后会在 helloworld.proto 同级目录下生成
# helloworld.pb.go 文件
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

#### main.go 文件
```
package main

import (
	"context"
	"flag"
	"fmt"
	"log"
	"net"

	"google.golang.org/grpc"
	pb "google.golang.org/grpc/examples/helloworld/helloworld"
)

var (
	port = flag.Int("port", 50051, "The server port")
)

// server is used to implement helloworld.GreeterServer.
type server struct {
	pb.UnimplementedGreeterServer
}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	log.Printf("Received: %v", in.GetName())
	return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main() {
	flag.Parse()
	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterGreeterServer(s, &server{})
	log.Printf("server listening at %v", lis.Addr())
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

#### gRPC接口调用
有很多插件或者工具，这里使用 `postman` 最新版的支持gRPC调用
```
Postman for Mac

Version 9.15.11

UI Version 9.15.11-ui-220404-0850

Desktop Platform Version 9.15.2

Architecture x64

OS Platform OS X 18.0.0
```

> postman使用方式就不介绍了，开箱即用

启动服务调用结果如下：
<img src="../styles/images/gRPC/call_gRPC.png" alt="call_gRPC" />

#### 参考文档：
- https://grpc.io/docs/what-is-grpc/