# ProtoBuf 介绍和运行时原理分析

## 1. 什么事ProtoBuf

Protocol Buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。

Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。

你可以定义数据的结构，然后使用特殊生成的源代码轻松的在各种数据流中使用各种语言进行编写和读取结构数据。你甚至可以更新数据结构，而不破坏由旧数据结构编译的已部署程序。

## 2. 序列化与反序列化

序列化和反序列化是我们日常开发中最常见的工作场景之一，它是我们通讯时采用的约定协议，如常见的网络协议（如HTTP、TCP等）；或是数据持久化，如固定的文件存储格式；或是安全中的加解密，如对称加密；等等。

- 序列化：将数据结构转化成二进制串的过程；
- 反序列化：将序列化后的二进制串转化成数据结构或对象的过程；

### 2.1 序列化协议的特性

- 通用性: 要求协议支持跨平台、跨语言；
- 健壮性：必需通过了足够的测试，足够健壮，才能保证序列化协议被人所接受和使用；
- 可扩展性：移动互联网时代，业务系统需求更新周期很快。所以要求序列号协议必须具有良好的可扩展性，支持自动增加新业务员字段；
- 性能：空间上的开销，序列化需要在原有的数据上加上描述字段，作为反序列化是解析所用。如果序列化过程的额外开销过高，会给网络、磁盘等带来压力；时间上的开销，如果序列化协议的解析时间过长，这样会使得序列化和反序列化成为整个系统的瓶颈；
- 安全性：为了信息安全，一般兼容现有的安全协议；如果实现新的安全协议，会提高开发；

### 2.2 序列化协议的组件

![](/Users/hawickhuang/Src/docs/learning-notes/base/grpc/components.jpeg)

- IDL文件：接口描述文件，参与通讯的各方需要对通讯的内容做相关的约定。这个接口描述语言，通常是与开发语言无关、平台无关的描述性语言。
- IDL编译器：编译器将IDL文件转换成各语言对应的动态库。
- Stub Lib：负责序列化和反序列化的工作代码。在客户端，接收用户参数，序列化后通过底层协议发送给服务端；或者接受服务端发送过来的序列化后的数据，经反序列化后返回的客户端程序；服务端与此相反；
- Client/Server：应用层程序代码。
- 底层协议栈和互联网：序列化之后的数据通过底层的传输层、网络层、链路层以及物理层协议转换成数字信号在互联网中传递。

Protocol Buffers（ProtoBuf）是一种平台无关、语言无关、可扩展的序列化结构数据的方法，主要用于网络上的数据交换和存储。



## 3. 常见序列化方法及对比

### 3.1 XML: 

XML是一种常用的序列化和反序列化协议，具有跨机器，跨语言等优点

### 3.2 JSON

 JSON起源于弱类型语言Javascript， 它的产生来自于一种称之为”Associative array”的概念，其本质是就是采用”Attribute－value”的方式来描述对象。 JSON的如下优点，使得它快速成为最广泛使用的序列化协议之一：

#### 3.2.1 优点

1、这种Associative array格式非常符合工程师对对象的理解。

2、它保持了XML的人眼可读（Human-readable）的优点。

3、相对于XML而言，序列化后的数据更加简洁。 来自于的以下链接的研究表明：XML所产生序列化之后文件的大小接近JSON的两倍。[xml与json比较](http://www.codeproject.com/Articles/604720/JSON-vs-XML-Some-hard-numbers-about-verbosity) 。

4、它具备Javascript的先天性支持，所以被广泛应用于Web browser的应用常景中，是Ajax的事实标准协议。

5、与XML相比，其协议比较简单，解析速度比较快。

6、松散的Associative array使得其具有良好的可扩展性和兼容性。

#### 3.2.2 典型应用场景和非应用场景

JSON在很多应用场景中可以替代XML，更简洁并且解析速度更快。典型应用场景包括：

1、公司之间传输数据量相对小，实时性要求相对低（例如秒级别）的服务。

2、基于Web browser的Ajax请求。

3、由于JSON具有非常强的前后兼容性，对于接口经常发生变化，并对可调式性要求高的场景，例如Mobile app与服务端的通讯。

4、由于JSON的典型应用场景是JSON＋HTTP，适合跨防火墙访问。

总的来说，采用JSON进行序列化的额外空间开销比较大，对于大数据量服务或持久化，这意味着巨大的内存和磁盘开销，这种场景不适合。没有统一可用的IDL降低了对参与方的约束，实际操作中往往只能采用文档方式来进行约定，这可能会给调试带来一些不便，延长开发周期。 由于JSON在一些语言中的序列化和反序列化需要采用反射机制，所以在性能要求为ms级别，不建议使用。

### 3.3 Thrift

Thrift是Facebook开源提供的一个高性能，轻量级RPC服务框架，其产生正是为了满足当前大数据量、分布式、跨语言、跨平台数据通讯的需求。 但是，Thrift并不仅仅是序列化协议，而是一个RPC框架。相对于JSON和XML而言，Thrift在空间开销和解析性能上有了比较大的提升，对于对性能要求比较高的分布式系统，它是一个优秀的RPC解决方案；但是由于Thrift的序列化被嵌入到Thrift框架里面，Thrift框架本身并没有透出序列化和反序列化接口，这导致其很难和其他传输层协议共同使用（例如HTTP）。

#### 3.3.1 典型应用场景和非应用场景

对于需求为高性能，分布式的RPC服务，Thrift是一个优秀的解决方案。它支持众多语言和丰富的数据类型，并对于数据字段的增删具有较强的兼容性。所以非常适用于作为公司内部的面向服务构建（SOA）的标准RPC框架。

不过Thrift的文档相对比较缺乏，目前使用的群众基础相对较少。另外由于其Server是基于自身的Socket服务，所以在跨防火墙访问时，安全是一个顾虑，所以在公司间进行通讯时需要谨慎。 另外Thrift序列化之后的数据是Binary数组，不具有可读性，调试代码时相对困难。最后，由于Thrift的序列化和框架紧耦合，无法支持向持久层直接读写数据，所以不适合做数据持久化序列化协议。

### 3.4. ProtoBuf

Protobuf具备了优秀的序列化协议的所需的众多典型特征：

1、标准的IDL和IDL编译器，这使得其对工程师非常友好。

2、序列化数据非常简洁，紧凑，与XML相比，其序列化之后的数据量约为1/3到1/10。

3、解析速度非常快，比对应的XML快约20-100倍。

4、提供了非常友好的动态库，使用非常简介，反序列化只需要一行代码。

Protobuf是一个纯粹的展示层协议，可以和各种传输层协议一起使用；Protobuf的文档也非常完善。由于其设计的理念是纯粹的展现层协议（Presentation Layer），目前并没有一个专门支持Protobuf的RPC框架。

#### 3.4.1 典型应用场景和非应用场景

Protobuf具有广泛的用户基础，空间开销小以及高解析性能是其亮点，非常适合于公司内部的对性能要求高的RPC调用。

由于Protobuf提供了标准的IDL以及对应的编译器，其IDL文件是参与各方的非常强的业务约束，另外，Protobuf与传输层无关，采用HTTP具有良好的跨防火墙的访问属性，所以Protobuf也适用于公司间对性能要求比较高的场景。

由于其解析性能高，序列化后数据量相对少，非常适合应用层对象的持久化场景。

Protobuf几乎支持所有流行的编程语言，几乎能用于我们当前所有项目。

它的主要问题在于其所支持的语言相对较少，另外由于没有绑定的标准底层传输层协议，在公司间进行传输层协议的调试工作相对麻烦。

## 4. 如何使用ProtoBuf

### 4.1 创建proto文件，定义数据结构

```protobuf
syntax = "proto3";

option go_package = "prototest.helloworld";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
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



### 4.2 使用protoc编译proto文件

```sh
protoc --go_out=. protobuf-test.proto
```

```go
// The request message containing the user's name.
type HelloRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
}
```

```go
// The response message containing the greetings
type HelloReply struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Message string `protobuf:"bytes,1,opt,name=message,proto3" json:"message,omitempty"`
}
```



### 4.3 使用stub接口，实现序列化和反序列化

### 4.3.1  服务端代码

```go
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

### 4.3.2 客户端代码

```go
var (
	addr = flag.String("addr", "localhost:50051", "the address to connect to")
	name = flag.String("name", defaultName, "Name to greet")
)

func main() {
	flag.Parse()
	// Set up a connection to the server.
	conn, err := grpc.Dial(*addr, grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewGreeterClient(conn)

	// Contact the server and print out its response.
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()
	r, err := c.SayHello(ctx, &pb.HelloRequest{Name: *name})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.GetMessage())
}
```

## 5. ProtoBuf数据类型

### 5.1 message

在proto中，所有结构化的数据都被称为 message。如3.1中，定义了两个message：`HelloRequest` 和`HelloReply`

### 5.2 分配字段号

每个消息定义中的每个字段都有**唯一的编号**。

这些字段编号用于标识消息在二进制格式中的字段，并且在使用消息类型后不应更改。

请注意，范围 1 到 15 中的字段编号需要一个字节进行编码，包括字段编号和字段类型。范围 16 至 2047 中的字段编号需要两个字节。所以你应该保留数字 1 到 15 作为非常频繁出现的消息元素。

可以指定的最小字段编号为1，最大字段编号为2^29^-1 或 536,870,911。也不能使用数字 19000 到 19999，因为它们是为 Protocol Buffers实现保留的。

### 5.3 保留字段（reserved）

通过使用`reserved`来标记删除或不再使用的字段，当任何用户试图使用这些字段标识符，Protocol Buffers 编译器将会报错。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

**注意，不能在同一个 `reserved` 语句中混合字段名称和字段编号**。如有需要需要像上面这个例子这样写。

### 5.4 各个语言标量类型对应关系

![](/Users/hawickhuang/Src/docs/learning-notes/base/grpc/lang_types.png)

### 5.5 枚举

在 message 中可以嵌入枚举类型。

```proto
message SearchRequest {
  string query = 1;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 2;
}
```

枚举类型需要注意的是，一定要有 0 值。

- 枚举为 0 的是作为零值，当不赋值的时候，就会是零值。
- 为了和 proto2 兼容。在 proto2 中，零值必须是第一个值。

另外在反序列化的过程中，无法被识别的枚举值，将会被保留在 messaage 中。因为消息反序列化时如何表示是依赖于语言的。在支持指定符号范围之外的值的开放枚举类型的语言中，例如 C++ 和 Go，未知的枚举值只是存储为其基础整数表示。在诸如 Java 之类的封闭枚举类型的语言中，枚举值会被用来标识未识别的值，并且特殊的访问器可以访问到底层整数。

在其他情况下，如果消息被序列化，则无法识别的值仍将与消息一起序列化。

### 5.6. 允许嵌套

Protocol Buffers 定义 message 允许嵌套组合成更加复杂的消息。

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

上面的例子中，SearchResponse 中嵌套使用了 Result 。

### 5.7. 更新 message

如果后面发现之前定义 message 需要增加字段了，这个时候就体现出 Protocol Buffer 的优势了，不需要改动之前的代码。不过需要满足以下规则：

-  不要改动原有字段的数据结构。

### 5.8 Map 类型

repeated 类型可以用来表示数组，Map 类型则可以用来表示字典。

```proto
map<key_type, value_type> map_field = N;

map<string, Project> projects = 3;
```

`key_type` 可以是任何 int 或者 string 类型(任何的标量类型，具体可以见上面标量类型对应表格，但是要除去 float、double 和 bytes)

**枚举值也不能作为 key**。

`key_type` 可以是除去 map 以外的任何类型。

需要特别注意的是 ：

- map 是不能用 repeated 修饰的。
- 线性数组和 map 迭代顺序的是不确定的，所以你不能依靠你的 map 是在一个特定的顺序。
- 为 `.proto` 生成文本格式时，map 按 key 排序。数字的 key 按数字排序。
- 从数组中解析或合并时，如果有重复的 key，则使用所看到的最后一个 key（覆盖原则）。从文本格式解析映射时，如果有重复的 key，解析可能会失败。

Protocol Buffer 虽然不支持 map 类型的数组，但是可以转换一下，用以下思路实现 maps 数组：

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

### 5.9 services

如果要使用 RPC（远程过程调用）系统的消息类型，可以在 `.proto` 文件中定义 RPC 服务接口，protocol buffer 编译器将使用所选语言生成服务接口代码和 stubs。所以，例如，如果你定义一个 RPC 服务，入参是 SearchRequest 返回值是 SearchResponse，你可以在你的 `.proto` 文件中定义它，如下所示：

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

与 protocol buffer 一起使用的最直接的 RPC 系统是 gRPC：在谷歌开发的语言和平台中立的开源 RPC 系统。gRPC 在 protocol buffer 中工作得非常好，并且允许你通过使用特殊的 protocol buffer 编译插件，直接从 `.proto` 文件中生成 RPC 相关的代码。

如果你不想使用 gRPC，也可以在你自己的 RPC 实现中使用 protocol buffers。您可以在 Proto2 语言指南中找到更多关于这些相关的信息。

还有一些正在进行的第三方项目为 Protocol Buffers 开发 RPC 实现。



## 6. ProtoBuf 的编码

### 5.1 TLV格式

Type-length-value（TLV）

T 字段表示报文类型，L 字段表示报文长度、V 字段往往用来存放报文的内容。

T、L 字段的长度往往固定 （ 通常为 1～4bytes ）

V 字段长度可变

#### 4.1.2 Varint

Varint 是一种紧凑的表示数字的方法。它用一个或多个字节来表示一个数字，值越小的数字使用越少的字节数。这能减少用来表示数字的字节数。

编码规则：

1. 对数字1进行varint编码后的结果为`0000 0001`，占用1个字节。相比`int32`的数字1存储占用4个字节，节省了3个字节的空间（主要是高位的0），而Varint的想法就是 **以标志位替换掉高字节的若干个0**。
2. 对数字300进行varint编码，原码形式为：

```
00000000 00000000 00000001 00101100
```

编码后的数据：第一个字节为`10101100`，第二个字节为`00000010`（不够7位高位补0）。

具体的变化步骤：

![number_300_varint.png](https://izualzhy.cn/assets/images/number_300_varint.png)

数值150，varint编码后的结果为`10010110 00000001`，即`0x96 0x01`。

根据上面的规则，摘抄了pb里对应的代码如下（`coded_stream.cc`）：

```
    static const int kMaxVarint32Bytes = 5;

    uint8 bytes[kMaxVarint32Bytes];
    int size = 0;
    while (value > 0x7F) {
      bytes[size++] = (static_cast<uint8>(value) & 0x7F) | 0x80;
      value >>= 7;
    }
    bytes[size++] = static_cast<uint8>(value) & 0x7F;
```

可以看到对于较大的数字(1«28)，使用的字节数由4个变成了5个，同时对于负数也有影响。pb里也有对应的解决方案。不过让我们先来解释下本文最开始编码字符`08 96 01`如何产生的问题。

varint编码希望以标志位能够节省掉高字节的0，但是负数的最高位一定是1， 所以varint在处理32位负数时会固定的占用5个字节。

#### 4.1.3 ZigZag

ZigZag是将有符号数统一映射到无符号数的一种编码方案，对于无符号数`0 1 2 3 4`，映射前的有符号数分别为`0 -1 1 -2 2`，负数以及对应的正数来回映射到从0变大的数字序列里，这也是”zig-zag”的名字来源。

详细的映射表是这样的：

| Signed Original | Encoded As |
| --------------- | ---------- |
| 0               | 0          |
| -1              | 1          |
| 1               | 2          |
| -2              | 3          |
| 2               | 4          |
| 2147483647      | 4294967294 |
| -2147483647     | 4294967295 |

对应的编码及解码方案(`wire_format_lite.h`)：

```
inline uint32 WireFormatLite::ZigZagEncode32(int32 n) {
  // Note:  the right-shift must be arithmetic
  return (n << 1) ^ (n >> 31);
}

inline int32 WireFormatLite::ZigZagDecode32(uint32 n) {
  return (n >> 1) ^ -static_cast<int32>(n & 1);
}

inline uint64 WireFormatLite::ZigZagEncode64(int64 n) {
  // Note:  the right-shift must be arithmetic
  return (n << 1) ^ (n >> 63);
}

inline int64 WireFormatLite::ZigZagDecode64(uint64 n) {
  return (n >> 1) ^ -static_cast<int64>(n & 1);
}
```

通过ZigZag编码后，就可以对负数进行varint编码了，abs比较小的负数转化成1个字节来存储。

## 6 反射

前面介绍过使用 ProtoBuf 的第一步便是创建 .proto 文件，定义我们所需的数据结构。但很多人没有意识到，这个过程同时也是为 ProtoBuf 提供我们数据元信息的过程，这些元信息包括数据由哪些字段构成，字段又属于什么类型以及字段之间的组合关系等。

当然元信息也并非一定由 .proto 文件提供，它也可来自于网络或其它可能的输入，只要它满足 ProtoBuf Message 的定义语法即可。那么元信息的可能来源和处理就有：

- .proto 文件

  - 使用 ProtoBuf 内置的工具 protoc 编译器编译，protoc 将 .proto 文件内容编码并写入生成的代码中（.pb.go 文件）
  - 使用 ProtoBuf 提供的编译 API 在运行时手动（指编码）解析 .proto 文件内容。实际上 protoc 底层调用的也正是这个编译 API。

- 非 .proto 文件

  - 从远程读取，如将数据与数据元信息一同进行 protobuf 编码并传输：

    

    ```protobuf
    message Req {
      optional string proto_file = 1;
      optional string data = 2;
    }
    ```

  - 从 Json 或其它格式数据中转换而来

  - ......

无论 .proto 文件来源于何处，我们都需要对其做进一步的处理，将其解析成内存对象，并构建其与实例的映射，同时也要计算每个字段的内存偏移。可总结出如下步骤：

1. 提供 .proto （范指 ProtoBuf Message 语法描述的元信息）
2. 解析 .proto 构建 FileDescriptor、FieldDescriptor 等，即 .proto 对应的内存模型（对象）
3. 之后每创建一个实例，就将其存到相应的实例池中
4. 将 Descriptor 和 instance 的映射维护到表中备查
5. 通过 Descriptor 可查到相应的 instance，又由于了解 instance 中字段类型（FieldDescriptor），所以知道字段的内存偏移，那么就可以访问或修改字段的值

## 7 性能

### 7.1 Benchmark

https://github.com/eishay/jvm-serializers/wiki/

#### 7.1.1 解析性能

![](/Users/hawickhuang/Src/docs/learning-notes/base/grpc/sertime_desertime.png)

#### 7.1.2 空间消耗

![](/Users/hawickhuang/Src/docs/learning-notes/base/grpc/compress_bytes.png)

## 8. 总结

ProtoBuf 是一个十分优秀的项目，在我们的项目中使用也非常广泛，其设计思想和源码对于我们的日常开发工作也提供了非常好的借鉴意义。本文只是做了一个基本的分析，如果大家想了解更多工程细节可进一步阅读 [ProtoBuf 源码](https://github.com/protocolbuffers/protobuf)。

参考文章：

1. [Protocol Buffers](https://developers.google.com/protocol-buffers/docs/overview)
1. [序列化和反序列化](https://tech.meituan.com/2015/02/26/serialization-vs-deserialization.html)
1. https://blog.csdn.net/zxhoo/article/details/53228303