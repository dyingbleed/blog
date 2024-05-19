---
title: "gPRC 学习笔记（一）：第一个 gPRC 服务"
date: 2023-06-28T21:49:59+08:00
draft: false
categories: ["dev"]
tags: ["go"]
---

[gRPC](https://grpc.io/) 官方网站的介绍：

> A high performance, open source universal RPC framework

gPRC 是由 Google 于 2015 年开源，在功能上参考了其内部 RPC 框架 Stubby

gPRC 构建在 HTTP/2 之上 [Protocal Buffer](https://protobuf.dev/) 编码格式

核心特性：

- 支持主流的编程语言
- 高效且简单的服务定义框架
- 支持双工流（Bi-directional Streaming）
- 可插拔的认证、追踪、负载均衡和健康检查

# 安装

macOS 使用 Brew 安装 Protocal Buffer 编译器 protoc：

```bash
brew install protoc
```

更新 PATH 环境变量：

```bash
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```

安装插件：

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

## Ping

接下来，基于 gPRC 实现一个简单的 Ping 服务，客户端和服务端均使用 Go 语言

安装 gRPC 库：

```bash
go get -u google.golang.org/grpc
```

创建 `proto/ping.proto` 文件：

```proto
syntax = "proto3";
package ping;

option go_package = "pkg/ping";

message Ping {
  int64 sequence = 1;
}

message Pong {
  int64 sequence = 1;
}

service PingService {
  rpc ping(Ping) returns (Pong);
}
```

客户端发送 Ping 消息，服务端返回 Pong 消息


生成代码：

```bash
protoc -I proto --go_out=. --go-grpc_out=. proto/*.proto
```

在 `pkg/ping` 目录下生成了两个文件 `ping.pb.go` 和 `ping_grpc.pb.go`


服务端代码 `server.go`：

```go
package main

import (
	"context"
	"google.golang.org/grpc"
	"log"
	"net"
)

import pb "dyingbleed.com/grpc-demo/pkg/ping"

type server struct {
	pb.UnimplementedPingServiceServer
}

func (s *server) Ping(ctx context.Context, in *pb.Ping) (*pb.Pong, error) {
	return &pb.Pong{Sequence: in.GetSequence()}, nil
}

func main() {
	s := grpc.NewServer()
	pb.RegisterPingServiceServer(s, &server{})
	lis, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatalf("fail to listen: %v", err)
	}
	if err = s.Serve(lis); err != nil {
		log.Fatalf("fail to serve: %v", err)
	}
}

```

客户端代码 `client.go`：

```go
package main

import (
	"context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"log"
	"time"
)

import pb "dyingbleed.com/grpc-demo/pkg/ping"

func main() {
	conn, err := grpc.Dial("127.0.0.1:8080", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("fail to dial: %v", err)
	}
	defer conn.Close()
	c := pb.NewPingServiceClient(conn)
	ctx := context.Background()
	sequence := time.Now().Unix()
	log.Printf("ping %d", sequence)
	pong, err := c.Ping(ctx, &pb.Ping{Sequence: sequence})
	if err != nil {
		log.Fatalf("fail to ping: %v", err)
	}
	log.Printf("pong %d", pong.GetSequence())
}

```