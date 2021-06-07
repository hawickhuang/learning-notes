# Proto3 协议讲解与使用教程

原文链接：[Language Guide (proto3)](https://developers.google.com/protocol-buffers/docs/proto3)



本指南旨在提供如何使用prtocol buffer语言来组织你的protocol buffer数据，包括.proto文件和如何从.proto文件生成数据访问类。

## 定义消息类型

首先我们来看一个非常简单的例子。假设你想定义一个查询请求的消息格式，每个请求会有一个查询字符串，结果的特定页面和每页的结果数目。下面是.proto文件的消息类型的定义：

```protobuf
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

* 文件的第一行指定你正在使用proto3语法，如果不指定，protocol buffer编译器默认使用ptoro2。这一行必须在文件第一行指定。（注释除外）
* SearchRequest消息定义了三个字段（键值对），即消息中应该包含的每个数据。每个字段都有名称和类型信息。

### 指定字段类型

在上面的例子中，所有的字段都是标量类型：整型（page_number和result_per_page）和字符串（query）。不过，你也可以定义更复杂的类型，包括枚举和其他数据类型。

### 分配字段编号

如你所见，消息定义中的每个字段都有一个唯一编号。这些编号用来在消息的二进制编码中标识字段，所以一经使用，就尽量不要修改它。1至15的编码需要一个1个字节，包含字段编码和字段类型信息；16至2047需要2个字节。因此，应该给那些经常出现的消息字段赋予1-15的值，也需要预留部分区域给后期很可能会添加进来的常用字段。

字段编码的最小值是1，最大值是2^29-1(536,870,911)。也不可以使用19000至19999之间的值（FieldDescriptor至kFirstReservedNumber)，它们被预留用于protobuf buffer的实现。当.proto文件出现这些值中的一个时，编译器会报错。同样，也不能使用任何以前的预留字段。

### 指定字段规则

消息中字段有两种形式：

* 单数：零个或一个该字段的值，但最多一个
* repeated：零个或多个该字段的值，值的顺序会被保留

proto3中，标量类型的repeated字段默认使用packed编码。

### 添加更多消息类型

在一个.proto文件中可以定义多个消息类型，这对于定义多个相关的消息很实用——例如，当你想要定义一个与SearchRequest对应的reply消息类型时，可以将它加入到同一个.proto文件中：

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```



### 添加注释

.proto文件的注释使用C/C++样式的语法：// 和 /* ... */

```protobuf
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```



### 预留字段

如果通过完全删除字段，或注释字段来更新消息，后面的用户在对消息进行更新时，就可能重新使用这些字段的编码。当导入相同.proto文件的旧版本时，就可能造成数据损坏、隐式bug或其它。有一种方法可以规避这种问题，就是将这些要弃用的字段编码(或名称，但也会导致json编码问题）设置为reserved。当再次使用这些编码时，protocol buffer的编译器会报错。

```protobuf
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

请注意，不能在同一个reserved语句里同时包含字段编码和字段名称。

### 你的.proto文件生成了什么？

当你对.proto文件运行protocol buffer编译器编译时，它会根据你选择的编程语言，生成文件中定义的消息类型、字段值的获取和设置、消息的序列化输出流和解析输入流里的消息相关的代码。

* C++，编译器生成 .h 和 .cc文件，包含文件中定义的消息类型的类代码
* Java，编译器生成.java文件，包含消息类型的类代码，消息类型类的Builder类
* Python，编译器生成一个包含每个消息类型的静态描述符，然后与元类一起，以提供运行时的python数据访问类
* Go，编译器生成.pb.go文件，包含每个消息类型的类型定义
* Ruby，编译器生成.rb文件，一个包含每个消息定义的ruby模块
* Objective-C，生成pbobjc.h和pbobjc.m文件，包含每个消息的类定义
* C#，生成 .cs 文件，包含每个消息的类定义
* Dart，生成.pb.dart文件，包含每个消息的类定义

你可以从你选择语言的教程中获取更多关于API使用的信息，若想获取更多的API的细节，可参考[API reference]()

## 标准数据类型

标量消息字段可以是下表中的任意一种，下表展示了在.proto文件的类型定义及其对应生成语言的类型：

| .proto   | Notes                                                                | C++    | Java       | Python      | Go      | Ruby                 | C#         | PHP            | Dart      |
| -------- | -------------------------------------------------------------------- | ------ | ---------- | ----------- | ------- | -------------------- | ---------- | -------------- | --------- |
| double   |                                                                      | double | double     | float       | float64 | Float                | double     | float          | double    |
| float    |                                                                      | float  | float      | float       | float32 | Float                | float      | float          | double    |
| int32    | 使用可变长度编码，对负数的编码效率低，若需要负数时，可使用sint32代替 | int32  | int        | int         | int32   | Fixnum/Bignum        | int        | integer        | Int       |
| int64    | 使用可变长度编码，对负数的编码效率低，若需要负数时，可使用sint64代替 | int64  | long       | int/long    | int64   | Bignum               | long       | integer/string | Int64     |
| uint32   | 使用可变长度编码                                                     | unit32 | int        | int/long    | uint32  | Fixnum/Bignum        | uint       | integer        | Int       |
| uint64   | 使用可变长度编码                                                     | uint64 | long       | Int/long    | uint64  | Bignum               | ulong      | integer/string | Int64     |
| sint32   | 使用可变长度编码，对负数编码效率比int32高                            | int32  | int        | int         | int32   | Fixnum/Bignum        | int        | integer        | Int       |
| sint64   | 使用可变长度编码，对负数编码效率比int64高                            | int64  | long       | int/long    | int64   | Bignum               | long       | integer/string | Int64     |
| fixed32  | 固定4个字节，若值经常大于2^28，编码效率比uint32高                    | uint32 | int        | int/long    | uint32  | Fixnum/Bignum        | uint       | integer        | Int       |
| fixed64  | 固定8个字节，若值经常大于2^56，编码效率比uint64高                    | uint64 | long       | int/long    | uint64  | Bignum               | ulong      | integer/string | Int64     |
| sfixed32 | 固定4字节                                                            | int32  | int        | int         | int32   | Fixnum/Bignum        | int        | integer        | Int       |
| sfixed64 | 固定8字节                                                            | int64  | long       | int/long    | int64   | Bignum               | long       | integer/string | Int64     |
| bool     |                                                                      | bool   | boolean    | bool        | bool    | TrueClass/FalseClass | bool       | boolean        | bool      |
| string   | 必须时UTF-8编码或7位ASCII编码                                        | string | String     | str/unicode | string  | String(UTF-8)        | string     | string         | String    |
| bytes    | 任意字节序列                                                         | string | ByteString | str         | []byte  | String(ASCII-8BIT)   | ByteString | string         | List<int> |

你可以从[Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding)中找到更多关于消息序列化时这些字段时如何编码的信息。

### 默认值

当转换消息时，如果被编码的消息中不包含特定的有效元素，则转换的对象的对应字段被赋予字段的默认值，这些默认值依据类型不同也会不同：

* 字符串类，默认值为空字符串
* 字节类，默认值为空字节
* bool类，默认值false
* 整型，默认值0
* 枚举，默认值为枚举的第一个值，也即0
* 消息体，默认值按语言不通，可参考[generated code guide](https://developers.google.com/protocol-buffers/docs/reference/overview)
* repeated，默认值为空，通常是空列表

请注意，当消息被解析后，没有办法去判定一个标量字段是否是显示设置为默认值的。例如，无法判断一个bool值是否被显式设置为false，在定义消息类型时要牢记在心。还有，如果你不想某个行为在默认情况下发生，那么就不应该在被设置为false时可以将该行为触发。还有当标量字段是默认值时，它不会被序列化到传输通道中。

参考[generated code guide](https://developers.google.com/protocol-buffers/docs/reference/overview)查看你对应语言对应默认值的处理细节。

### 枚举

在定义消息类型时，你可能希望有个字段有一个预定义的值列表。例如，假设你想给SearchRequest请求添加一个corpus字段，corpus的值可以是UNISERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS 或 VIDEO。你可以通过添加enum到你的消息定义中来实现。

在下面的例子中，我们添加了一个Corpus枚举和它所有可能的值，另外添加一个corpus字段到SearchRequest消息类型中：

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

如你所见，Corpus枚举的第一个常数值是0，每个枚举定义的第一个常数值都必须是0，这是因为：

* 必须包含0值，我们才可以用0作为默认值
* 0值必须是第一个值，为了与proto2兼容，因为proto2的第一个值就是默认值

你可以为不同的枚举常量指定相同的值来定义别名，要实现这种功能你需要将allow_alias选项设置为true，否则，protocol buffer编译器将会产生错误。

```protobuf
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

枚举常量必须在32位整型范围内，因为数据传输时，enum使用varint编码，负数的效率会比较低，所以不推荐使用负数。你可以在消息体内定义枚举，也可以在消息体外，在消息体外定义的枚举可以在整个.proto文件中使用。你还可以使用语法：MessageType.EnumType来实现在一个消息体中定义另一个消息体中的枚举的字段类型。

当编译一个包含枚举的.proto文件时，会为C++或Java生成对应的enum，为python生成特定的EnumDescriptor类，可在运行时创建包含一系列符号常量的类。

在反序列化期间，无法识别的消息会被保留，但如何表现则依据与语言。在支持可以超出指定范围值的开放枚举类型的语言中，如C++和Go，未知枚举值简单的存储为其基础整数值。对于封闭式枚举类型语言（如Java），枚举中的一个对象将用来表示未知枚举值，也可以通过特定的访问器来访问其基础整数。任何情况下，不可识别的值都会被序列化到消息中。

你可以参考[generated code guide](https://developers.google.com/protocol-buffers/docs/reference/overview)来获取更多关于各种语言如何处理枚举消息的细节。

## 其他数据类型

你可以使用其他消息类型作为一个字段类型。例如，假设你想在每个SearchResponse消息中包含Result消息，你可以在SearchResponse的.proto文件中，添加Result消息的定义，并在SearchResponse中定义一个Result类型的字段。

```protobuf
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### 导入

在上面的例子中，Result消息类型跟SearcchResponse消息类型定义在同一个.proto文件，如果想使用在其他.proto文件中定义的消息类型呢？

你可以通过导入其他.proto文件来使用它，只需要在文件头部添加一条import语句就好了：

```protobuf
import "myproject/other_protos.proto";
```

默认情况，只能使用导入过的.proto文件中的定义。但是，有时你需要移动.proto文件到新的位置。现在你可以在旧位置放一个虚拟的.proto文件，使用import public的概念，将所有的导入转发到新位置，而不需要直接移动.proto文件，也不需要一次修改所有调用网站。导入包含import public语句的proto文件都可以传递import public依赖项。例如：

```protobuf
// new.proto
// All definitions are moved here
```

```protobuf
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```

```protobuf
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

在protocol编译器中，可以通过-I或—proto_path选项来指定编译器对导入文件的搜索路径，如果未设置，编译器会搜索它的执行路径。通常，应该把—proto_path设置为项目的根目录，所有的import语句使用相对路径。

### 嵌套

你可以在消息类型中定义或使用其它消息类型，在下面这个例子中，Result消息类型被定义在SearchResponse消息中：

```protobuf
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

如果想在这个消息类型的父级消息类型外使用它，你可以使用 Parent.Type模式：

```proto
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

你可以随意嵌套消息类型：

```protobuf
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

### 更新消息类型

如果现有的消息类型无法满足你的需求，例如，你希望消息格式中有一个新的其它字段，但你希望仍然使用旧格式创建的代码，无需担心，你不需要破坏已有代码即可简单更新消息类型。只需记住以下规则：

* 不要修改已有字段的字段序列值
* 如果添加了新字段，新代码依旧可以解析旧消息类型。你应该记住这些字段的默认值，以便新代码可以与旧消息类型正常交互。同样，新定义的消息类型也可以被旧代码解析，旧代码会忽略这些新的字段。
* 只要保证字段序列号不会再使用，那么更新消息类型时就可以移除字段。你也可能希望重命名字段，或者添加前缀”OBSOLETE_“，或者可以将字段序列号设置为reserved，就可以避免后面意外重用了这些序列号。
* int32，uint32，int64，uint64和bool都是兼容的，意味着你可以将这些类型相互替换，也能保持向前向后兼容。如果从流中解析的值与对应的类型不一致，这会得到与C++处理类型转换时相同的结果（一个64位整数被转换为int32时，会被截断为32位）
* sint32和sint64彼此兼容，但与其它整数类型不兼容
* 只要bytes是有效的UTF-8，那么string与bytes是兼容的
* 如果bytes包含消息编码版本，则bytes与嵌入消息兼容
* fixed32与sfix32兼容，fixed64与sfixed64兼容
* enum 与int32，uint32，int64和uint64兼容，当值字节不同时，会被截断。但请注意，在反序列化消息时，客户端代码会做不同的处理。例如，不可识别的proto3的enum类型会被保留在消息中，但反序列化后如何表示则依语言而不同，整型字段都会保留它们的值。
* 将单个值设置为新的oneof的成员是安全和二进制兼容的。当可以保证不会有代码一次设置多个值时，将多个字段加入新的oneof中也是安全的。将任何字段加入已有的oneof中是不安全的。

### Unknown字段

未知字段指的是protocol buffer在序列化数据时，解析器无法识别的字段。例如，当使用旧协议文件来解析包含新字段的新协议文件生成的信息时，旧协议文件认为那些新字段为未知字段。

起初，proto3版本会丢弃未知字段，但在3.5版本中，我们重新与proto2版本保持了一致，保留了未知字段。在3.5及以后的版本中，未知字段会被保留，并包含在序列化输出中。

### Any

Any消息类型允许你在没有对应的.proto文件时，在定义时使用它的消息类型。Any类型会被序列化为bytes，同时含有一个表示该消息类型的URL作为全局唯一标识符。若要使用Any类型，需要：import `google/protobuf/any.proto`

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

给的消息类型的默认类型URL为：**type.googleapis.com/*packagename*.messagename**

不同语言支持持运行时库来实现安全的pack和unpack Any类型值。例如Java，Any类型会有特定的pack()和unpack()访问器，在C++，会有PackForm()和UnpackTo()函数：

```java
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

用于支持处理Any类型的运行时库当前还在开发中。

若你熟悉proto2语法，Any类型代替了其extensions。

### Oneof

如果你有一个包含很多字段的消息类型，而且在同一时刻最多只能设置其中一个字段，那么你可以使用oneof来强制这种行为，还可以借此节省内存。

oneof字段与常规字段有两个不同，一是oneof中的所有字段共享内存，二是在同一时刻最多只能设置一个字段。对oneof中字段的设置会清除所有其它成员的值。你可以通过特定的case（）或WhichOneof（）来查看哪个区字段被设置，这依赖于你的语言。

#### Oneof使用

在.proto文件中定义oneof时，只需要在oneof后面添加oneof类型对应的名称来实现，如下面例子：

```protobuf
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

你可以在oneof中嵌套使用oneof，也可以使用任何除repeated以为的任何类型字段。

在生成的代码中，oneof字段与其它字段一样，包含setters和getters。还会有一个特定的方法来检查oneof中那个字段被赋值了。可以参阅 [API reference](https://developers.google.com/protocol-buffers/docs/reference/overview)查看相关细节。

#### Oneof的特点

* 设置oneof字段的值会自动清除oneof中其它属性的值。所以如果你设置了几个oneof中的字段，那么只有最后的字段的设置才会生效。

  ```c++
  SampleMessage message;
  message.set_name("name");
  CHECK(message.has_name());
  message.mutable_sub_message();   // Will clear name field.
  CHECK(!message.has_name());
  ```

* 如果解析器在输入流中看到同一个oneof的多个成员，只有最后一个成员将会被解析到消息流中

* oneof不能有repeated成员

* Reflection APIs对oneof字段有效

* 当使用C++时，请确保代码不存在内存崩溃问题。下面的代码示例会导致内存崩溃：因为调用set_name()会删除sub_message

  ```c++
  SampleMessage message;
  SubMessage* sub_message = message.mutable_sub_message();
  message.set_name("name");      // Will delete sub_message
  sub_message->set_...            // Crashes here
  ```

* 当使用C++时，Swap() oneof类型的两个消息，每条消息将会包含另外的那条消息。在下面这个例子中，msg1将会有sub_message，而msg2将会有name

  ```c++
  SampleMessage msg1;
  msg1.set_name("name");
  SampleMessage msg2;
  msg2.mutable_sub_message();
  msg1.swap(&msg2);
  CHECK(msg1.has_sub_message());
  CHECK(msg2.has_name());
  ```

#### 向后兼容问题

在添加或移除oneof字段时要特别小心，如果在坚持oneof的值时返回None/NOT_SET，它可能意味着oneof字段未赋值，或者它赋值了，但oneof的版本不一样。这两种情况无法区分，因为无法区分输入流中的未知字段是否为oneof成员。

##### 标签重用问题

* 添加或移除oneof成员：在序列化和解析消息后，可能会丢失一些信息(会清除一些字段)。不过，你可以安全地添加一个字段到新的oneof中，如果你知道那个成员被设置，也可以一次性添加多个oneof的成员。
* 删除oneof成员后加回：在消息序列化和解析后，当前oneof的设置值会被清除
* 分割或合并oneof：这与移动常规字段的效果一样

### Maps

如果你希望定义关联映射的数据，protoco buffer提供了便捷语法

```protobuf
map<key_type, value_type> map_field = N;
```

 上面的key_type可以时整型或者字符串，除浮点数和bytes外标量类型均可以。请注意，enum类型不可以时key_type，value_type可以是除map外的任意类型

例如，你希望创建一个protjects的字典类型，每个Project消息由一个字符串键指定，你可以这样定义：

```protobuf
map<string, Project> projects = 3;
```

* Map字段不可以时repeated
* Map在输出流和迭代时顺序不确定，所以你不能期望map的元素有特定的顺序
* 生成.proto文件的文本格式时，maps按键排序，整数键按值排序
* 当从输入流或者合并情况下，map中存在重复的键，只有最后一个有效。从文本模式解析map时，重复的键会导致解析失败
* 如果在map中只设置的key，而没有设置值，在序列化的行为依语言而定。在C++/Java/Python中，值类型默认值会被序列化，在其它语种，将不序列化该键和值

目前，生成map的API已被所有proto3支持的语言提供，你可查阅[API reference](https://developers.google.com/protocol-buffers/docs/reference/overview)获取更多细节

#### 向后兼容

Map语法与下面的定义等效，所以，对于protocol buffer实现语言不支持map的，依然可以实现相同功能

```protobuf
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

任何支持map的protocol buffer实现语言，必须同时支持生成和接受上述定义

### Packages

你可以添加一个可选的package定义到.proto文件的开头，以避免协议消息类型名字的冲突

```protobuf
package foo.bar;
message Open { ... }
```

你可以在你的消息类型前加上包名来使用它

```protobuf
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

包定义对生成代码的影响取决于你的语言

* C++，生成的类会在一个C++ namespace内，例如，Open在foo::bar命名空间内
* Java，包会成为一个Java包，除非你显式在.proto文件添加 option java_package语句
* Python，包定义会被忽略，因为Python的模块由它们在文件系统的组织决定
* Go，包定义会作为Go包名，除非你在.proto文件显式定义 option go_package
* Ruby，生成的类在嵌套的命名空间内，并按需转换为Ruby大小写模式：首字母大写，如果首个字符不是字母，则添加PB_前缀。例如。Open 将会在 Foo::Bar命名空间内
* C#，在转换为PascalCase后，会作为命名空间，除非你在.proto文件中显式定义option csharp_namespace语句。例如，Open会在 Foo.Bar 命名空间内

#### 包和名称解析

protocol buffer语言的类型名称解析与C++类似，首先搜索最内部那层，然后搜索下一层。每个包都被认为是它父级包的“内部”。一个"."开头的（如.foo.bar.Baz)表示从最外层开始。

protocol buffer编译器通过解析所有导入的.proto文件，来处理类型名称。每个语言的代码生成器知道如何将此一一对应，即使它们的范围规则不一致

## 定义服务

如果你想将消息类型应用到RPC（远程过程调用）系统中，你可以在.proto文件中添加RPC服务接口，protocol buffer编译器将会按你选择的语言生成对应的接口和stub。例如，定义一个RPC服务，需要输入SearchRequest和返回SearchResponse参数，如下

```:antigua_barbuda:
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}

```

与protocol buffer最合适的RPC系统是gRPC，谷歌开放的语言和平台重力的开源RPC系统。gRPC特别适用于protocol buffer，你可以通过特定的protocol buffer编译插件，从.proto文件中生成对应的RPC代码。

若你不想用gRPC，protocol buffer也可以用于自定义的RPC系统中，可以参阅 [Proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto#services)获取更多细节。

当前还有很多第三方的项目，基于protocol buffer来实现RPC。你可以参阅[third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)来了解这些项目信息。

## Json Mapping

proto3支持标准Json编码，这使系统间的数据交互更加容易。下表逐个类型地介绍了它的编码。

如果在Json编码中某个值缺失或者是null，在它被解析为protocol buffer时，将会用默认值表示。如果字段在protocol buffer中有默认值，在Json编码中该字段会被忽略以节省空间。你可以通过设置来实现在Json编码中显示默认值。

| proto3               | JSON          | JSON example                           | Notes                                                                                                                                                                                                         |
| -------------------- | ------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| message              | object        | {"fooBar": v, "g": null, ...}          | 生成json对象。消息字段的名称将会转换为lowerCamelCase后作为json对象的key。如果段指定来json_name，则指定值将作为json键。解析器可以接受lowerCamelCase或原始proto字段名称。null是所有字段类型的可接受值和默认值。 |
| enum                 | string        | “FOO_BAR”                              | 使用proto中enum的名称。解析器接受enum名称和整数                                                                                                                                                               |
| map<K,V>             | object        | {"k": v, ...}                          | 所有的key都转换为strings                                                                                                                                                                                      |
| repeated V           | array         | [v, ...]                               |                                                                                                                                                                                                               |
| string               | string        | "Hello World"                          |                                                                                                                                                                                                               |
| bytes                | base64 string | "YWJjMTIzIT8kKiYoKSctPUB+"             | 将使用带填充的标准base64编码的字符串。标准或URL安全的base64编码都是可接受的。                                                                                                                                 |
| Int32,fixed32,uint32 | number        | 1, -10, 0                              | json值是一个十进制数字。整型和字符串都是可接受的                                                                                                                                                              |
| Int64,fixed64,uint64 | string        | "1", "-10"                             | 将使用带填充的标准base64编码的字符串。标准或URL安全的base64编码都是可接受的。                                                                                                                                 |
| bool                 | true, false   | true, false                            |                                                                                                                                                                                                               |
| Float,double         | number        | 1.1, -10.0, 0, "NaN", "infinity"       | json值可以是一个整数或者是"NaN","Infinity","-Infinity"。数字和字符串都可接受，指数表示法也可接受                                                                                                              |
| Any                  | object        | {"@type": "url", "f": v, ...}          | 如果Any包含有特殊Json映射值，它会被转换为：{"@type": xxx, "value": yyy}，否则，它会被转换为json对象，其中“@type”字段会是数据的真实类型                                                                        |
| Timestamp            | string        | "1972-01-01T10:00:20.021Z"             | 使用RFC3339，其中生成的输出将始终被Z标准化并使用0,3,6或9个小数位。也接受“Z”以外的偏移。                                                                                                                       |
| Duration             | string        | "1.000340012s", "1s"                   | 生成的输出始终包含0,3,6或9个小数位，具体取决于所需的精度，后跟后缀“s”。接受任何小数位（或是没有），只要它们符合纳秒精度并且后缀“s”是必需的。                                                                  |
| Struct               | object        | { … }                                  | 任何JSON对象。请参阅struct.proto。                                                                                                                                                                            |
| Wrapper types        | 各种类型      | 2, "2", "foo", true,"true", null, 0, … | Wrappers在JSON中使用与包装基元类型相同的表示形式，除了在数据转换和传输期间允许和保留null。                                                                                                                    |
| FieldMask            | string        | "f.fooBar,h"                           | 请参考field_mask.proto                                                                                                                                                                                        |
| ListValue            | array         | [foo, bar, …]                          |                                                                                                                                                                                                               |
| Value                | value         |                                        | 任何JSON对象。                                                                                                                                                                                                |
| NullValue            | null          |                                        | JSON null                                                                                                                                                                                                     |

#### Json选项

proto3的json实现可提供以下定制功能

* 以默认值填充字段：proto3的json输出中默认不显示值为默认值的字段。可以设置选项来覆盖此行为，从而在输出中以默认值显示
* 忽略未知字段：proto3 json解析器默认拒绝未知字段，不过解析时可以设置选项来忽略未知字段
* 使用字段名代替lowerCamelCase名：proto3的json输出默认情况下支持将字段名转换为lowerCamelCase名。可以设置选项来指定使用字段名作为json的键名。proto3的json解析器必须要同时支持这两种格式的字段名
* 将enum作为整数而不是字符串：在默认情况下，json输出使用enum的名称。可以设置选项来指定使用enum的整数值。

### 选项

.proto文件中的各项声明可以使用很多选项来添加评注。这些评注不会改变声明的意义，但会影响它在特定上下文中的处理方式。可以从google/protobuf/descriptor.proto中查看完整的选项定义。

对于文件级选项，它应该写在文件的顶级域，而不是在消息、枚举或服务定义里。对于消息级选项，应该写在消息定义里。对于字段级选项，应该写在字段定义里。选项当然也可以写在enum类型、enum值、service类型、service方法中，但当前这方面中还没什么有用的选项。

下面是一些最常用的选项：

* java_package(文件级选项)：希望生成的java类的包。如果不显式定义java_package，默认使用.proto文件的package值。然后proto的包名作为java的包名并不符合java的体验，java使用反向域名。如果不是生成java代码，该选项无作用

  ```:bar_chart:
  option java_package = "com.example.foo";
  ```

* Java_multiple_files(文件级选项)：在包的顶级定义消息、枚举和服务，而不是后文说的在外部类中

  ```protobuf
  option java_multiple_files = true;
  ```

* Java_outer_classname(文件选项)：生成的java代码中最外层的类名(和文件名)。如果没有显式指定该选项的值，最外层类名将会由.proto文件的名字的驼峰表示法替代。对于非java代码该选项无效。

  ```:antigua_barbuda:
  option java_outer_classname = "Ponycopter";
  ```

* optimize_for(文件选项)：可以被设置为：SPEED, CODE_SIZE和 LIFE_RUNTIME，它会以以下方式影响生成C++和Java代码生成器（或者包括第三方生成器）：

  * SPEED(默认)：protocol buffer编译器将会生成包括序列化、解析和执行其他操作消息类型的代码。这些代码都经过高度优化

  * CODE_SIZE：protocol buffer编译器将生成最小的类和机遇共享、基于反射的代码来实现序列化、解析和其他的操作。生成的代码会比SPEED小的多，但也会慢很多。它也会与SPEED模式生成同样的公共API。这个模式在拥有大量.proto文件和不需要它们太快的处理速度的程序中更有用

  * LITE_RUNTIME：protocol buffer编译器将会生成基于“lite”运行时库的类(libprotobuf-lite，而不是libprotobuf)。“lite”运行时库比完整的库小很多，约小一个数量级，但省略了描述符和反射等特定功能。这对移动端程序特别有用。它仍然会实现与SPEED模式同样的API接口。生成的类仅会实现MessageLite接口，它只是Message接口的一个子集

    ```:antigua_barbuda:
    ption optimize_for = CODE_SIZE;
    ```

* cc_enable_arenas(文件级选项)：为C++代码开启arena分配

* objc_class_prefix(文件级选项)：给.proto文件生成的Objective-C类和enum添加前缀。该选项没有默认值，你应该参照[recommended by Apple](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4)设置3-5个大写字母。请注意，两个字母的前缀都是Apple保留的。

* deprecated(文件级选项)：如果设置为true，表示该字段已经弃用，新代码不应该再使用。在大多数语言中该选项无实际效果。在Java，这会添加@Deprecated修饰。将来，其他特定语言生成器可能会给字段访问器生成弃用修饰，这将会编译使用弃用代码时产生告警。如果字段不再使用，而且想避免新用户使用它，应该使用保留语法来替代弃用语法

  ```:antigua_barbuda:
  int32 old_field = 6 [deprecated=true];
  ```

#### 自定义选项

protocol buffer还允许你定义和使用自己的选项。这是大部分人不需要的高级功能。如果你需要创建自定义选项，可参考 [Proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto.html#customoptions) 获取更多细节。注意创建自定义选项使用扩展。

### 生成你的类

要生成Java, Python, C++, Go, Ruby, Objective-C或 C#代码，你需要在.proto文件定义消息类型，还需要使用protoc编译器对.proto文件进行编译。如果你尚未安装编译器，下载[安装包](https://developers.google.com/protocol-buffers/docs/downloads.html)，参照README.me文件进行安装。对于Go，还需要安装特定的代码生成器插件，你可以在[golang/protobuf](https://github.com/golang/protobuf/)找到该插件的安装说明。

protocol buffer编译器的使用方法如下：

``` protobuf
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

* IMPORT_PATH指定导入路径目录，编译器会在该目录查找.proto文件。如果省略，将会使用当前路径。可以通过多次使用—proto_path来指定多个导入目录；编译器会按顺序搜索。-I = IMPORT_PATH是—proto_path的简短用法

* 你可以指定一个或多个输出目录：

  * —cpp_out：生成C++代码的目录
  * --java_out：生成Java代码的目录
  * --python_out：生成Python代码的目录
  * --go_out：生成Go代码的目录
  * --ruby_out：生成Ruby代码的目录
  * --objc_out：生成Objective-C代码的目录
  * --csharp_out：生成C#代码的目录
  * --php_out：生成PHP代码的目录

  另外一个方便之处，当 DST_DIR 以 .zip 或 .jar结尾时，编译器会将输出写入 zip格式文件中，.jar还会依据Java JAR的定义给出一个清单文件。请注意，如果输出文件依存在，它将会被覆盖；编译器还不能实现添加文件到一个已存在的存档文件中。

* 必须提供一个或多个.proto文件作为输入，多个.proto文件也可以一次定义。虽然文件是依据于当前目录命名的，但是每个文件必须在某一个IMPORT_PATH中，这样编译器才能确定其规范名称。