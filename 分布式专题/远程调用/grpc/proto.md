## Proto3指南

### 定义消息类型

```protobuf
syntax = "proto3";

message OrderRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

`syntax` 指定使用`proto3`语法，如果不指定的话，协议缓冲区编译器会使用proto2编译。这必须是文件的第一行、非注释行。

`OrderRequest` 消息指定了三个字段，每个字段都有一个名称和类型。



### 定义字段类型

分为标量类型和复合类型。

- 标量类型详见[标量值类型](#index)
- 复合类型其中包括枚举和其它消息类型。



### 分配字段编号

消息定义中的每个字段都有**唯一的编号**。这些编号的作用是在**消息二进制格式中标识字段**，一旦有一个消息类型被使用，就不应更改。

- 1-15之间的字段编号占用一个字节进行编码，包括字段编号和字段类型。
- 16-2047之间的字段编号占用两个字段。

因此，非常频繁出现的消息元素保留数字 1 到 15，为将来可能添加的频繁出现的元素留出一些空间。



> 注意：如果在开发中修改了客户端或者服务端其中一个端中某个字段的编号，那么会导致传递该字段值时为空，需要特别注意，修改字段需确保两个端所使用的字段编号一致。



规定：最小字段编号为1，最大字段编号为536,870,911，但是19000～19999之间的数字编号不能在proto被使用。因为它们是Protocol Buffer实现保留的。



### 指定字段规则

消息字段可以是以下之一：

- singular：默认字段规则，在格式正确的消息有零个或一个该字段。
- `repeated`：在格式正确的消息中重复任意次数，包括零次。

在proto3中，标量数值类型的`repeated`字段默认使用`packed`编码，为了提高高效的编码效率，应尽可能使用选项[packed=true]，例如。

```protobuf
syntax = "proto3";

message OrderRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  repeated int32 arr = 4 [packed=true];
}
```



### 添加注释

使用C/C++的风格。

```protobuf
/* xxx
 * xxx */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```



### 保留字段

如果通过删除和注释某个字段来更新消息类型，其它开发者对类型进行更新时，依旧可以使用被删除或注释的某个字段，如果以后将代码进行还原，则会导致严重的问题，包括数据损坏、隐私错误等。

为了解决这种问题，我们可通过`reserved`指定已删除字段的字段编号或者字段名称，如果其它开发者尝试使用这些字段编号则会编译报错。

```protobuf
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
  int32 keys = 4;
  reserved 4; // 将字段编号为4作为保留字段，使其字段不能被使用
  reserved result_per_page;
  reserved 3 to max;
}
```



注意：同一个`reserved`语句不能同时使用字段名称和字段编号。【测试字段名称好像不行】



### 标量值类型{#index}

| .proto Type | Java Type  |                             备注                             |
| :---------: | :--------: | :----------------------------------------------------------: |
|   double    |   double   |                                                              |
|    float    |   float    |                                                              |
|    int32    |    int     | 使用可变长度编码，负数编码效率低下。如果您的字段可能具有负值，请改用sint32。 |
|    int64    |    long    | 使用可变长度编码，负数编码效率低下。如果您的字段可能具有负值，请改用sint64。 |
|   uint32    |    int     |                      使用可变长度编码。                      |
|   uint64    |    long    |                      使用可变长度编码。                      |
|   sint32    |    int     | 使用可变长度编码，有符号的int值。与常规int32相比，它们更有效地编码负数。 |
|   sint64    |    long    | 使用可变长度编码，有符号的int值。与常规int64相比，它们更有效地编码负数。 |
|   fixed32   |    int     |    始终为四个字节。如果值通常大于2^28，则比uint32更有效。    |
|   fixed64   |    long    |    始终为八个字节。如果值通常大于2^56，则比uint64更有效。    |
|  sfixed32   |    int     |                       始终为四个字节。                       |
|  sfixed64   |    long    |                       始终为八个字节。                       |
|    bool     |  boolean   |                                                              |
|   string    |   String   |         字符串必须始终包含UTF-8编码或7位ASCII文本。          |
|    bytes    | ByteString |                     可以包含任意字节序列                     |

在Protocol Buffer Encoding序列化消息时，可以找到有关类型如何编码的信息。

- 在java中，无符号32位和64位整型被表示成他们的整型对应形式，最高位被储存在标志位中。
- 对于所有的情况，设定值会执行类型检查以确保此值是有效。
- 64位或者无符号32位整型在解码时被表示成为ilong，但是在设置时可以使用int型值设定，在所有的情况下，值必须符合其设置其类型的要求。
- Integer在64位的机器上使用，string在32位机器上使用。

对于原始类型，不生成hasXXX方法，对于原始类型如果想要使用hasXXX方法，需要使用[wrappers.proto](https://github.com/google/protobuf/blob/master/src/google/protobuf/wrappers.proto)里的类型。



### 默认值

解析消息时，如果编码的消息不包含特定的singular元素，则解析对象中的相应字段将设置为该字段的默认值。

- 对于字符串，默认值为空字符串。
- 对于字节，默认值为空字节。
- 对于bool，默认值为false。
- 对于数字类型，默认值为0。
- 对于枚举，默认值为第一个定义的枚举值，必须为0。
- 重复字段的默认值为空列表。
- 对于消息类型（message），没有被设置，和语言有关【java中是一个未进行setter的对象】。



> 注意：对于标量消息字段，一旦解析了消息，就无法判断字段是否明确设置为默认值。

例如：不应该定义boolean的默认值为false作为任何行为的触发方式，如果一个标量类型字段被设置为标记位，这个值不应该被序列化传输。



### 枚举

在下面的例子中，在消息格式中添加了一个叫做Corpus的枚举类型【它含有所有可能的值】，以及一个类型为Corpus的字段：

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
  }
  Corpus corpus = 4;
}
```

如你所见，Corpus枚举的第一个常量映射为0，每个枚举类型必须将其第一个类型映射为0，这是因为：

- 必须有一个0值，我们可以用这个0值作为默认值。
- 这个零值必须为第一个元素，为了兼容proto2语义，枚举类的第一个值总是默认值。



可以通过为不同的枚举常量分配相同的值来定义别名，需要将`allow_alias`选项设置为true，否则将会编译报错。

正确方式：

```protobuf
message MyMessage1 {
  enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
  }
}
```

错误方式：

```protobuf
message MyMessage2 {
  enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
  }
}
```

 标量类型必须在32位整数值的范围内，因为枚举类型采用的是可变编码方式，对负数不够高效，不推荐在枚举中使用负数。



在反序列化期间，未识别的枚举值将保留在消息中，尽管在反序列化消息时如何表示取决于语言。在支持值超出指定符号范围的public枚举类型的语言（例如 C++ 和 Go）中，未知枚举值只是作为其底层整数表示存储。在 Java 等具有private枚举类型的语言中，枚举中的 case 用于表示无法识别的值，并且可以使用特殊访问器访问底层整数。在任何一种情况下，如果消息被序列化，无法识别的值仍将与消息一起序列化。



### 使用其它消息类型

案例一：

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

案例二：

```protobuf
enum EnumAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
  }
message MyMessage3 {
	EnumAllowingAlias alias = 0;
}
```

案例三：

```protobuf
message MyMessage2 {
  enum EnumAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
  }
}

message MyMessage3 {
	MyMessage2.EnumAllowingAlias alias = 0;
}
```



### 更新消息类型（*）

如果现有的消息类型不再满足您的所有需求，例如，您希望消息格式有一个额外的字段，但您仍然希望使用以旧格式创建的代码，在不破坏任何现有代码的情况下更新消息类型非常简单。请记住以下规则：

- 不要更改任何现有字段的字段编号。
- 如果添加新字段，使用“旧”消息格式的代码序列化的任何消息仍可以由新生成的代码解析，你应该记住这些元素的默认值，以便新代码可以与旧代码生成的消息正确交互。
- 只要在更新的消息类型中不再使用字段编号，就可以删除字段。您可能想要重命名该字段，也许添加前缀“OBSOLETE_”，或者保留字段编号，以便您的未来用户`.proto`不会意外地重复使用该编号。
- `int32`、`uint32`、`int64`、`uint64`和`bool`都兼容 - 这意味着您可以将字段从这些类型中的一种更改为另一种，而不会破坏向前或向后的兼容性。如果从连线中解析出的数字不适合相应类型，您将获得与在 C++ 中将该数字强制转换为该类型相同的效果（例如，如果将 64 位数字读取为 int32，它将被截断为 32 位）。
- `sint32`并且`sint64`彼此兼容，但与其他整数类型不兼容。
- `string`并且`bytes`只要字节是有效的 UTF-8 就兼容。
- `bytes`如果字节包含消息的编码版本，则嵌入的消息兼容。
- `fixed32`与兼容`sfixed32`，并`fixed64`用`sfixed64`。
- 对于`string`、`bytes`和 消息字段，`optional`与 兼容`repeated`。给定重复字段的序列化数据作为输入，`optional`如果它是原始类型字段，则期望此字段的客户端将采用最后一个输入值，如果它是消息类型字段，则合并所有输入元素。请注意，这**不是**一般的数值类型，包括布尔变量和枚举安全。数字类型的重复字段可以在打包格式中序列化，在`optional`预期字段时不会正确解析。
- `enum`与`int32`, `uint32`, `int64`, 和`uint64`线格式兼容（请注意，如果它们不适合，值将被截断）。但是请注意，当消息被反序列化时，客户端代码可能会以不同的方式对待它们：例如，无法识别的 proto3`enum`类型将保留在消息中，但是当消息被反序列化时如何表示取决于语言。Int 字段始终只保留其值。
- 将单个值更改为**新** 值的成员`oneof`是安全且二进制兼容的。`oneof`如果您确定没有代码一次设置多个，则将多个字段移入一个新字段可能是安全的。将任何字段移动到现有字段中`oneof`都是不安全的。



### Map

如果你希望创建一个关联映射，protocol buffer提供了一种快捷的语法：

```protobuf
map<key_type,value_type> map_field = N;
```

其中key_type可以是Integer和String类型（除了浮点型和字节型的任意标量类型），value_type可以是任意类型。

- Map字段不能是`repeated`。
- 序列化后的顺序和map迭代器的顺序是不确定的，所以你不要期望以固定顺序处理Map。
- 当为.proto文件产生生成文本格式的时候，map会按照key 的顺序排序，数值化的key会按照数值排序。
- 从序列化中解析或者融合时，如果有重复的key则后一个key不会被使用，当从文本格式中解析map时，如果存在重复的key。

### 包

可以为.proto文件新增一个可选的package声明符，用来防止不同的消息类型有命名冲突。

```protobuf
package foo.bar;
message Open { 

}
```

可以在定义消息类型的字段时使用包说明符。

```protobuf
message Foo {
  foo.bar.Open open = 1;
}
```



### 定义服务

如果您想在 RPC（远程过程调用）系统中使用您的消息类型，可以在`.proto`文件中定义一个 RPC 服务接口，协议缓冲区编译器将以您选择的语言生成服务接口代码和存根。例如，

```protobuf
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

最直观的使用protocol buffer的RPC系统是gRpc一个由谷歌开发的语言和平台中的开源的PRC系统，gRPC在使用protocol buffer时非常有效，如果使用特殊的protocol buffer插件可以直接为您从.proto文件中产生相关的RPC代码。



请求参数为空在GRPC中如何定义呢？

google protobuf已经提供了空参数

```protobuf
google.protobuf.Empty
```

本质上就是

```protobuf
message Empty {}
```

例子：

```protobuf
import "google/protobuf/empty.proto";
service SearchService {
  rpc Search(google.protobuf.Empty) returns (SearchResponse);
}
```



### 选项

在定义.proto文件时能够标注一系列的options。Options并不改变整个文件声明的含义，但却能够影响特定环境下处理方式。完整的可用选项可以在google/protobuf/descriptor.proto找到。



- 文件级别的，意味着它可以作用于最外范围，不包含在任何消息内部、enum或服务定义中；
- 消息级别的，意味着它可以用在消息定义的内部；
- 有些选项可以作用在域、enum类型、enum值、服务类型及服务方法中。



常见的选项：

- `java_package` (文件选项) :这个选项表明生成java类所在的包。如果在.proto文件中没有明确的声明java_package，就采用默认的包名。当然了，默认方式产生的 java包名并不是最好的方式，按照应用名称倒序方式进行排序的。如果不需要产生java代码，则该选项将不起任何作用。如：`option java_package="com.example.foo"`
- `java_outer_classname` (文件选项): 该选项表明想要生成Java类的名称。如果在.proto文件中没有明确的java_outer_classname定义，生成的class名称将会根据.proto文件的名称采用驼峰式的命名方式进行生成。如（foo_bar.proto生成的java类名为FooBar.java）,如果不生成java代码，则该选项不起任何作用。如：`option java_outer_classname="Ponycopter"`
- `optimize_for`(文件选项): 可以被设置为 SPEED、CODE_SIZE和LITE_RUNTIME。这些值将通过如下的方式影响C++及java代码的生成：
  - `SPEED (default)`: protocol buffer编译器将通过在消息类型上执行序列化、语法分析及其他通用的操作。这种代码是最优的。
  - `CODE_SIZE`: protocol buffer编译器将会产生最少量的类，通过共享或基于反射的代码来实现序列化、语法分析及各种其它操作。采用该方式产生的代码将比SPEED要少得多， 但是操作要相对慢些。当然实现的类及其对外的API与SPEED模式都是一样的。这种方式经常用在一些包含大量的.proto文件而且并不盲目追求速度的应用中。
  - `LITE_RUNTIME`: protocol buffer编译器依赖于运行时核心类库来生成代码（即采用libprotobuf-lite 替代libprotobuf）。这种核心类库由于忽略了一 些描述符及反射，要比全类库小得多。这种模式经常在移动手机平台应用多一些。编译器采用该模式产生的方法实现与SPEED模式不相上下，产生的类通过实现 MessageLite接口，但它仅仅是Messager接口的一个子集。
- `deprecated`(字段选项):如果设置为`true`则表示该字段已经被废弃，并且不应该在新的代码中使用。在大多数语言中没有实际的意义。在java中，这回变成`@Deprecated`注释，在未来，其他语言的代码生成器也许会在字标识符中产生废弃注释，废弃注释会在编译器尝试使用该字段时发出警告。如果字段没有被使用你也不希望有新用户使用它，尝试使用保留语句替换字段声明。如：`int32 old_field = 6[deprecated=true];`

 

Github：https://github.com/hjforever/grpc-demo-for-java

Protohub：https://developers.google.com/protocol-buffers/docs/proto3