ProtoBuf 

## 1. 什么是ProtoBuf

序列化和反序列化是我们日常开发中最常见的工作场景之一，它是我们通讯时采用的约定协议，如常见的网络协议（如HTTP、TCP等）；或是数据持久化，如固定的文件存储格式；或是安全中的加解密，如对称加密；等等。

- 序列化：将数据结构转化成二进制串的过程；
- 反序列化：将序列化后的二进制串转化成数据结构或对象的过程；

### 1.1 序列化协议的特性

- 通用性：
- 健壮性：
- 可扩展性：
- 性能：
- 安全性：

### 1.2 序列化协议的组件

- IDL文件
- IDL编译器
- Stub Lib：
- Client/Server：
- 底层协议栈和互联网：

Protocol Buffers（ProtoBuf）是一种平台无关、语言无关、可扩展的序列化结构数据的方法，主要用于网络上的数据交换和存储。



## 2. 常见序列化方法及对比

- JSON
- XML
- Thrift
- ProtoBuf

## 3. 如何使用ProtoBuf

### 3.1 创建proto文件，定义数据结构

### 3.2 使用protoc编译proto文件

### 3.3 使用stub接口，实现序列化和反序列化

## 4. ProtoBuf 的编码

### 4.1 编码结构

#### 4.1.1 TLV格式

#### 4.1.2 Varints 

#### 4.1.3 ZigZag

#### 4.1.4 Varint

#### 4.1.5 bool、enum

#### 4.1.6 sint32、sint64

#### 4.1.7 64bit、32bit

#### 4.1.8 fixed64、sfixed64、double、fixed32、sfixed32、float

#### 4.1.9 Length-delimited 





参考文章：

1. [序列化和反序列化](https://tech.meituan.com/2015/02/26/serialization-vs-deserialization.html)