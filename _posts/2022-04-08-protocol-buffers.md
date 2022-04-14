---
layout: post
title:  Protocol Buffers 相关知识简记
date:   2022-04-08 10:30:00 +0800
categories: Protocol Buffers
tag: note
---

* content
{:toc}

需要科学上网工具：[官网传送门](https://developers.google.com/protocol-buffers/)

本篇文章主要是来源于官网，避免作者个人曲解误人子弟，有不理解或者疑问最好直接阅读官方文档，这里主要也是我个人做一些简单的阅读笔记，以下主要是学习 proto3

#### What are protocol buffers?

**官方解释如下**
> Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

**NOTE**
> 谷歌出品用于支持跨平台、跨语言、可扩展的序列化结构数据
类似于xml，但是更小、更快、更简单
定义一次数据结构，编译之后在不同语言等环境复用

#### 支持的语言

**来源官方**
> Protocol buffers currently support generated code in Java, Python, Objective-C, and C++. With our new proto3 language version, you can also work with Kotlin, Dart, Go, Ruby, and C#, with more languages to come.

**NOTE**
> proto3版本开始支持Kotlin, Dart, Go, Ruby, and C#

#### How do I start
1. [Download](https://github.com/protocolbuffers/protobuf) and install the protocol buffer compiler.
2. Read the [overview](https://developers.google.com/protocol-buffers/docs/overview).
2. Try the tutorial for your chosen language.


#### protocol buffers 的一些优点

**来源官方**
> - Compact data storage
> - Fast parsing
> - Availability in many programming languages
> - Optimized functionality through automatically-generated classes

#### Language Guide (proto3)

以下内容摘自于[docs-proto3](https://developers.google.com/protocol-buffers/docs/proto3)

##### Defining A Message Type

```
// 非注视的第一行定义使用proto3语法，否则默认为proto2
syntax = "proto3";

// 定义一个请求的消息结构，包含一个条件，和分页条件
message SearchRequest {
  string query = 1; // 查询参数，从1开始
  int32 page_number = 2; // 页码参数
  int32 result_per_page = 3; // 单页参数
}
```

**原文**
> As you can see, each field in the message definition has a unique number. These field numbers are used to identify your fields in the message binary format, and should not be changed once your message type is in use. Note that field numbers in the range 1 through 15 take one byte to encode, including the field number and the field's type (you can find out more about this in Protocol Buffer Encoding). Field numbers in the range 16 through 2047 take two bytes. So you should reserve the numbers 1 through 15 for very frequently occurring message elements. Remember to leave some room for frequently occurring elements that might be added in the future.

The smallest field number you can specify is 1, and the largest is 2^29 - 1, or 536,870,911. You also cannot use the numbers 19000 through 19999 (FieldDescriptor::kFirstReservedNumber through FieldDescriptor::kLastReservedNumber), as they are reserved for the Protocol Buffers implementation - the protocol buffer compiler will complain if you use one of these reserved numbers in your .proto. Similarly, you cannot use any previously reserved field numbers.

**NOTE**
> protobuf 有2个版本，默认版本是 proto2，如果需要 proto3，则需要在非空非注释第一行使用 syntax = "proto3" 标明版本。
package，即包名声明符是可选的，用来防止不同的消息类型有命名冲突。
消息类型 使用 message 关键字定义，Student 是类型名，name, male, scores 是该类型的 3 个字段，类型分别为 string, bool 和 []int32。字段可以是标量类型，也可以是合成类型。
每个字段的修饰符默认是 singular，一般省略不写，repeated 表示字段可重复，即用来表示数组类型。
每个字符 =后面的数字称为标识符，每个字段都需要提供一个唯一的标识符。标识符用来在消息的二进制格式中识别各个字段，一旦使用就不能够再改变，标识符的取值范围为 [1, 2^29 - 1] 。
.proto 文件可以写注释，单行注释 //，多行注释 /* ... */
一个 .proto 文件中可以写多个消息类型，即对应多个结构体(struct)。

##### Scalar Value Types

|  .proto Type   | Notes  | C++ Type | Java/Kotlin Type | Python Type | Go Type | Ruby Type | C# Type | PHP Type | Dart Type |
|  ----          |  ----  |  ----    | ----             |  ----       |   ----  |     ----  | ----    |    ----  | ----      |  
| double         |        |  double  | double           | float       | float64 | Float     | double  | float    | double    |
| float          |        |  float   | float            | float       | float32 | Float     | double  | float    | double    |
| int32          | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead.	|  int32   | int           | int         | int32   | Fixnum or Bignum (as required)     | int  | integer    | int    |
| int64          |   Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead.     |  int64   | long            | int/long       | int64 | Bignum     | long  | integer/string    | int64    |
| uint32        |    Uses variable-length encoding.    |  uint32   | int            | int/long       | uint32 | Fixnum or Bignum (as required)     | uint  | integer    | int    |
| uint64        |    Uses variable-length encoding.    |  uint64   | long           | int/long       | uint64 | Bignum     | ulong  | integer/string    | int64    |
| sint32          | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s.	|  sint32   | int           | int         | int32   | Fixnum or Bignum (as required)     | int  | integer    | int    |
| sint64          |   Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s.     |  int64   | long            | int/long       | int64 | Bignum     | long  | integer/string    | int64    |
| fixed32        |   Always four bytes. More efficient than uint32 if values are often greater than 2^28.    |  uint32   | int            | int/long       | uint32 | Fixnum or Bignum (as required)     | uint  | integer    | int    |
| fixed64        |    Always eight bytes. More efficient than uint64 if values are often greater than 2^56.    |  uint64   | long           | int/long       | uint64 | Bignum     | ulong  | integer/string    | int64    |
| sfixed32          | Always four bytes.	|  int32   | int           | int         | int32   | Fixnum or Bignum (as required)     | int  | integer    | int    |
| sfixed64          | Always four bytes.  |  int64   | long            | int/long       | int64 | Bignum     | long  | integer/string    | int64    |
| bool         |        |  bool  | boolean           | bool       | bool | TrueClass/FalseClass     | bool  | boolean    | bool    |
| string         |   A string must always contain UTF-8 encoded or 7-bit ASCII text, and cannot be longer than 2^32.     |  string  | String           | str/unicode       | string | String(UTF-8)     | string  | string    | String    |
| bytes         |    May contain any arbitrary sequence of bytes no longer than 2^32.	    |  string  | ByteString           | str (Python 2) bytes (Python 3)       | []btye | String (ASCII-8BIT)	 | ByteString  | string    | List    |

##### Default Values
When a message is parsed, if the encoded message does not contain a particular singular element, the corresponding field in the parsed object is set to the default value for that field. These defaults are type-specific:

- For strings, the default value is the empty string.
- For bytes, the default value is empty bytes.
- For bools, the default value is false.
- For numeric types, the default value is zero.
- For enums, the default value is the first defined enum value, which must be 0.
- For message fields, the field is not set. Its exact value is language-dependent. See the generated code guide for details.

**NOTE**
> 对于标量消息字段，一旦解析消息，就无法判断字段是显式设置为默认值（例如，布尔值是否设置为false）还是根本没有设置

##### Enumerations

```
message MyMessage1 {
  enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
  }
}
```

**NOTE**
> 枚举类型的第一个选项的标识符必须是0，这也是枚举类型的默认值；
别名（Alias），允许为不同的枚举值赋予相同的标识符，称之为别名，需要打开allow_alias选项。

##### Using Other Message Types

Result是另一个消息类型，在 SearchReponse 作为一个消息字段类型使用。

```
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

也可以导入引用其它文件
```
import "myproject/other_protos.proto";
```

##### Nested Types

上述消息还可以使用嵌套
```
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

外部message依旧可以使用message内的定义
```
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

支持多层嵌套
```
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

##### Updating A Message Type

> If an existing message type no longer meets all your needs – for example, you'd like the message format to have an extra field – but you'd still like to use code created with the old format, don't worry! It's very simple to update message types without breaking any of your existing code. Just remember the following rules:

- Don't change the field numbers for any existing fields.
- If you add new fields, any messages serialized by code using your "old" message format can still be parsed by your new generated code. You should keep in mind the default values for these elements so that new code can properly interact with messages generated by old code. Similarly, messages created by your new code can be parsed by your old code: old binaries simply ignore the new field when parsing. See the Unknown Fields section for details.
- Fields can be removed, as long as the field number is not used again in your updated message type. You may want to rename the field instead, perhaps adding the prefix "OBSOLETE_", or make the field number reserved, so that future users of your .proto can't accidentally reuse the number.
- int32, uint32, int64, uint64, and bool are all compatible – this means you can change a field from one of these types to another without breaking forwards- or backwards-compatibility. If a number is parsed from the wire which doesn't fit in the corresponding type, you will get the same effect as if you had cast the number to that type in C++ (for example, if a 64-bit number is read as an int32, it will be truncated to 32 bits).
- sint32 and sint64 are compatible with each other but are not compatible with the other integer types.
- Embedded messages are compatible with bytes if the bytes contain an encoded version of the message.
- For string, bytes, and message fields, optional is compatible with repeated. Given serialized data of a repeated field as input, clients that expect this field to be optional will take the last input value if it's a primitive type field or merge all input elements if it's a message type field. Note that this is not generally safe for numeric types, including bools and enums. Repeated fields of numeric types can be serialized in the packed format, which will not be parsed correctly when an optional field is expected.
- enum is compatible with int32, uint32, int64, and uint64 in terms of wire format (note that values will be truncated if they don't fit). However be aware that client code may treat them differently when the message is deserialized: for example, unrecognized proto3 enum types will be preserved in the message, but how this is represented when the message is deserialized is language-dependent. Int fields always just preserve their value.
- Changing a single value into a member of a new oneof is safe and binary compatible. Moving multiple fields into a new oneof may be safe if you are sure that no code sets more than one at a time. Moving any fields into an existing oneof is not safe.

##### Unknown Fields
> Unknown fields are well-formed protocol buffer serialized data representing fields that the parser does not recognize. For example, when an old binary parses data sent by a new binary with new fields, those new fields become unknown fields in the old binary.

Originally, proto3 messages always discarded unknown fields during parsing, but in version 3.5 we reintroduced the preservation of unknown fields to match the proto2 behavior. In versions 3.5 and later, unknown fields are retained during parsing and included in the serialized output.

##### Any

Any 可以表示不在 .proto 中定义任意的内置类型，比如go的泛型
```
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```
##### Oneof

> To define a oneof in your .proto you use the oneof keyword followed by your oneof name, in this case test_oneof:

```
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

**NOTE**
>设置oneof字段将自动清除oneof字段的所有其他成员。因此，如果您设置了几个oneof字段，则只有最后一个字段仍然有值。

##### Maps

```
map<key_type, value_type> map_field = N;
```

```
syntax = "proto3";
message Product
{
    string name = 1; // 商品名
    // 定义一个k/v类型，key是string类型，value也是string类型
    map<string, string> attrs = 2; // 商品属性，键值对
}
```

**NOTE**
> key_type可以是任何整数或字符串类型（除浮点类型和字节之外的任何标量类型）。请注意，枚举不是有效的key_type;
> value_type 可以是除另一个映射之外的任何类型。

##### Packages

> You can add an optional package specifier to a .proto file to prevent name clashes between protocol message types.

```
package foo.bar;
message Open { ... }
```

```
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

##### Defining Services

如果消息类型是用来远程通信的(Remote Procedure Call, RPC)，可以在 .proto 文件中定义 RPC 服务接口。例如我们定义了一个名为 SearchService 的 RPC 服务，提供了 Search 接口，入参是 SearchRequest 类型，返回类型是 SearchResponse

```
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

##### JSON Mapping
> Proto3 supports a canonical encoding in JSON, making it easier to share data between systems. The encoding is described on a type-by-type basis in the table below.

|  proto3   | JSON  | JSON example |
|  ---   | ---  |  ---   |
|  message  | object  |  {"fooBar": v, "g": null, …}   |
|  enum     | string  |  "FOO_BAR"   |
|  map<K,V> | object  |  {"k": v, …} |
|  repeated V | array  |  [v, ...]   |
|  bool   | true, false  |  true, false   |
|  string   | string	  |  Hello World!"	   |
|  string   | base64 string	  |  "YWJjMTIzIT8kKiYoKSctPUB+"	   |
|  ...   | ...	  |  ...	   |

> 参考官网

##### Options
比如java_package (file option): The package you want to use for your generated Java/Kotlin classes. If no explicit java_package option is given in the .proto file, then by default the proto package (specified using the "package" keyword in the .proto file) will be used. However, proto packages generally do not make good Java packages since proto packages are not expected to start with reverse domain names. If not generating Java or Kotlin code, this option has no effect.

```
option java_package = "com.example.foo";
```

##### Generating Your Classes

将 `.proto` 文件编译成你所使用的编程语言
```
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

- IMPORT_PATH：导入路径指定要查找 `.proto` 文件的目录。解析导入指令时的原始文件。如果省略，则使用当前目录。通过多次传递--proto_path选项，可以指定多个导入目录；他们将按顺序被搜索。-I=_IMPORT_PATH_可以用作--proto_PATH的缩写形式
- --cpp_out：generates C++ code in DST_DIR （类似的，--java_out：java，--kotlin_out，--python_out，--go_out，--ruby_out，--objc_out，--csharp_out，--php_out）
- 必须提供一个或多个.proto文件作为输入。可以一次指定多个.proto文件。尽管这些文件是相对于当前目录命名的，但每个文件都必须驻留在IMPORT_PATH导入的其中一个路径中，以便编译器可以确定其规范名称。
