---
layout: post
title:  go protobuf 简单的例子
date:   2022-04-15 10:30:00 +0800
categories: Protocol Buffers
tag: note
---

* content
{:toc}

> 需要科学上网

[gotutorial文档](https://developers.google.com/protocol-buffers/docs/gotutorial)

#### 文件目录结构

```
go-protobuf
├── protobuf     
│   ├── addressbook.pb.go
├──	addressbook.proto
└── main.go     
```

#### 编写 proto 文件

```
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";

option go_package = "github.com/protocolbuffers/protobuf/examples/go/tutorialpb";

message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
```


#### 编译

```
protoc -I=. --go_out=./protobuf ./addressbook.proto --go_opt=paths=source_relative
```

**PS:** 编译器插件protoc-gen-go将安装在 中 $GOBIN，默认为$GOPATH/bin. 它必须在您$PATH的协议编译器protoc中才能找到它，参考上一篇

#### main.go 

```
package main

import (
	"fmt"
	pb "go-protobuf/protobuf"
)

func main()  {
	person := pb.Person{
		Id:    12345,
		Name:  "上山打老虎",
		Email: "hello@example.com",
		Phones: []*pb.Person_PhoneNumber{
			{Number: "86-54321", Type: pb.Person_HOME},
		},
	}

	fmt.Printf("person: %+v", person)
}
```