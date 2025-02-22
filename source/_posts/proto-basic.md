---
title: protobuf 基础入门
date: 2019-11-17 23:10:48
tags: protobuf
categories:
- 技术
- 其他
---
## `protobuf` 是什么

`protobuf` 全名是 `protocolbuffers` 是一种序列化数据编码的方式, 由 Google 开发, 特点是语言无关, 平台无关, 高效率.
你可以在[github](https://github.com/protocolbuffers/protobuf) 和 [官网](https://developers.google.com/protocol-buffers) 了解更多详细信息.
你可以想象成 `json` 编码解码, 不过效率高很多. 编码后的文件是二进制格式, 无法直接读取阅读.
与 `json` 不同的是, 你需要预先定义数据的结构.
`protobuf` 有版本2 和 版本3, 我们这里就仅介绍版本3.
<!-- more -->

## 一个简单的例子

我们来定义一个简单的 response 结构, 假设用来返回api 的调用结果.
里面需要有两个字段, `code`, 和 `err_msg`, 
`code` 用来表示返回码, 我们规定200 表示成功, 其他表示错误.
`err_msg` 用来表示错误的详情.
如果用json 来表示一个特定的返回那么是这样的

``` json
{
    "code": 200,
    "err_msg" "ok"
}
```

和 `json` 的自由不同的是, 我们必须要预先定义我们的数据结构.
先直接上代码

``` protobuf
// 这里是注释
/* 这里也是注释*/
// 空行会被忽略 

syntax = "proto3";

message SimpleResponse {
    int32 code = 1;
    string err_msg = 2;
}
```

`syntax = "proto3";` 表示你使用 版本3 的protobuf, 对于版本3, 这是必选的. 而且除去空行和注释之外, 这必须是第一行.
后面的则表示我们定义了一个消息结构体, 名字是 `SimpleResponse`(可以看到结构体名称的命名方式是驼峰式命名规则)
里面有两个字段, 第一个字段是 `int32` 类型的, 名字是 `code`. 第二个是 `string` 类型的字段, 名称是 `err_msg`(可以看到字段的命名规则是下划线分割的规则);
这可能对于若类型语言开发者不太友好, 不过这也是高效率与小体积的关键. 比如我们现在需要解码一个数据, 在知道字段类型的情况下, 我们可以直接读取前面32位直接作为 `code` 的值(实际上并不是如此, int32 也不总是32位, 只是为了方便理解), 省去了字符串转换位数字的步骤, 也减少了文件体积.
除了这两个类型还有其他很多预定义的字面值, 详细列表可以看[这里](https://developers.google.com/protocol-buffers/docs/proto3#scalar), 当然, 也支持自定义的类型或是数组.
最后的 `=1; =2` 并不是默认值的意思. 我们需要为每一个字段定一个id, 在同一个结构体里面, 每个字段都必须唯一, 是用来解析, 优化数据用的.
一旦定义好, 就不要随意修改了. 比如我们一开始像上面这样定义 `SimpleResponse`, 然后后面又将两者调换id, 本来前32为是 `code` 的, 现在前32 位变成了 `err_msg` 的一部分, 解析自然会出问题.
需要注意的是, 前面15个id(id 从1 开始编号)分配给常用字段, 如果以后可能会扩充字段, 最好留一些空间给以后用, 主要和编码优化有关.

## 复合类型

上面的例子中我们都是用的字面值, protobuf 还支持自定义类型, 数组等

### 数组

使用数组的关键字是 `repeated`, 语法是 `repeated type filedname = N;`
例如:

``` protobuf
message FilerListReposne {
    repeated File files = 1;
}
```

这样我们就可以在`files` 字段中存放多个`File` 类型的数据了

### hash 表, 字典

如果需要定义一个哈希表, 或者在某些语言中称为字典, 散列表的数据结构, 你一定会用到 `map` 关键字
定义方式和c++的模版类似, 语法为 `map<key_type, value_type> filedname = N;`
例如

``` protobuf
message File {
    string path = 1;
    map<string, string> extend_info = 2;
}
```

注意, key_type 有一定限制, 可以用字面值类型中的整数类型或是string, 需要注意, 枚举不能用作key_type
value_type 只要不是另一个map 都可以.
map 本身不能作为数组的元素, 也就你不能这样写 `repeated map<string, string> filed = 1`
和其他语言一样, map 是无序的.

## 关于继承

看到这里, 你是否想要问`protobuf` 是否支持继承. 答案是否定的, `protobuf` 不支持继承.
不管你喜欢还是不喜欢, 总之没有, 你可以使用类似的方式替换, 例如使用组合而不是继承.
例如我们常常使用这种结构来表示一系列我们的API返回结果:

``` c++
// 例如, 我们有一组文件返回的api. 通常我们会这么编写数据结构
// BaseResponse 包含api 的基础调用状态
class BaseResponse {
    // 0 : success, other failed
    int32 code;
    // error reason
    string message;
}
// 文件上传返回
class FileUploadResponse: BaseResponse {
    // 成功后的文件hash
    string hash;
    // 文件大小
    int32 size;
}
// 获取文件信息返回
class FileInfoResposne: BaseResponse {
    // 文件路径
    string path;
    // 文件大小
    int32 size;
    // 文件hash
    string hash;
}
```

这一套在 protobuf 不好使了.
只能改改思路, 例如使用组合而不是继承.

``` protobuf
// 错误状态
message Error {
    int32 code = 1;
    string msg = 2;
}
// 文件上传返回
message FileUploadResponse {
    bool succeed = 1;
    Error error = 2;
    string hash = 3;
    int32 size = 4;
}
// 获取文件信息返回
class FileInfoResposne: BaseResponse {
    bool succeed = 1;
    Error error = 2;
    string path = 3;
    int32 size = 4;
    string hash = 5;
}
```

总之, 暂时是没有继承的, 如果你确实需要, 可以取给他们的项目提 `feature` 或是自己 `fork` 一个来实现.

## 枚举类型

枚举类型可以看作是一堆有关联的, 有名字的常量.
`protobuf`支持定义枚举类型, 让你的数据更有意义.
例如, 你要定义一个3状态的布尔变量用来存储数据.

``` protobuf
enum TripleBool {
    UNDEFINED = 0; // 未知状态
    FALSE = 1;
    TRUE = 2;
}
```

可以看到, 我的枚举类型以0 开始, 事实上枚举类型总是, 而且**必须**以 0 开始, 每个值都需要手动设定值.
不过可以允许重复值, 需要加上一个配置项 `allow_aliase = true`, 例如

``` protobuf
enum TripleBool {
    option allow_aliase = true;
    UNDEFINED = 0;
    FALSE = 1;
    NO = 1;
    TRUE = 2;
    YES = 2;
}
```

枚举类型实际是一个32位整数, 小心不要溢出. 
另外, 虽然实际上可以用负数(**枚举值的第一个值必须是0, 不能是其他值**), 不过由于性能并不会很好, 所以没有特殊情况的时候, 不要轻易使用.
枚举值除了单独定义, 也可以在一个消息结构体里面定义作为内联类型, 如

``` protobuf
message BasicResponse{
    enum TripleBool {
        UNDEFINED = 0;
        FALSE = 1;
        TRUE = 2;
    }
    TripleBool succeed = 1; // 使用时, 像普通类型使用即可.
    string msg = 2;
}
message LoginResponse{
    // 内联的枚举也可以在其他消息结构体中使用. 使用 . 分隔结构体名称与枚举名称即可
    BasicResponse.TripleBool succeed = 1; 
    string token = 2;
}
```

### 枚举类型解析错误的处理

将解析数据时, 如果某一个枚举字段解析后没有对应取值时(例如, 上面的 `TripleBool` 例子中, 解析到的数据中存储的实际数据是100), `protobuf` 将会想办法将原始数据(100)以某种方式保存起来, 具体方案和语言有关.
c++/go 这种枚举其实只是一个整数的, 就将整数值原始值存起来.
java 这种只能选择特定枚举值的, 解析将值设置为未定义/空值, 可以另外的特殊方法获取原始值.
任何情况下, 数据都不会丢失, 重新序列化为数据时, 原始值又会重新存起来.

## 任意类型

如果你还没想好使用什么数据类型, 可以使用预设的`Any` 类型凑合用用.
例如

``` protobuf
import "google/protobuf/any.proto";

message Response {
  int32 code = 1;
  google.protobuf.Any content = 2;
}
message ImageContent {
  string path = 1;
  int32 image_width = 2;
  int32 image_height = 3;
}
message AudioContent {
  string path = 1;
  float length = 2;
}
```

import 表示引入一个文件, 具体的使用可以下面会介绍.
`google.protobuf` 是包名, 具体下面再详细介绍.
在 `protobuf` 编译后, `Any` 会提供 `pack` 和 `unpack` 方法以供拆包和解包.
具体语言不同, 函数名称又些许差异.
例如c++ 使用方法方式中如下.

``` c++
Response response;
if (response.content().Is<ImageContent>()) {
    ImageContent image;
    detail.UnpackTo(&image);
    //.... now handle image
  }
} else if (response.content().Is<AudioContent>()) {
    AudioContent audio;
    detail.UnpackTo(&audio);
    //.... now handle audio
  }
}
```

## oneof 联合体

有些时候, 在同一个消息中, 根据状态的不同, 可能返回不同类型的数据, 但是如果将所有数据同时并列写出来, 可能对内存不太友好. 我们可以考虑使用 `oneof` 关键字定义联合体
例如

``` protobuf 
message Response {
    int32 code = 1;
    oneof ResponseDetail {
        string error_message = 0;
        string data;
    }
}
```

注意`oneof` 中不支持数组类型.
如果对oneof 中不同的字段多次赋值, 只有最后一次赋值有效.
如果你不确定具体设置的是什么数据, 可以使用 has_xxx 函数检测.
如果你使用c++, 有些重要的地方需要额外关注, 否则可能会导致崩溃, [参考这里](https://developers.google.com/protocol-buffers/docs/proto3#oneof-features)

## 默认值

所有值的默认值都是 0 值, 具体和类型相关.

* 字符串就是空字符串.
* 布尔值就是false
* 数字就是 0
* 枚举就是第一个 0 值.
* 二进制类型就是一个空数据.
* 数组就是一个空数组.
* 复合类型则每一个内部值都取 0 值.
  
解析时会自动缺少给缺少的字段填充默认值. 同样, 一个设定为 0 值的字段在序列化时也会被自动忽略/
需要注意的是, 你并不能区别 一个值为 0 的字段是故意设定为 0 还是没有填充导致(例如你通过读取到一个bool 字段为 false, 你不能判断是用户想要设置为false, 还是没有填写这个字段).

## 引入其他预定义的文件

某些文件可能会经常被反复使用, 例如我们上面定义的 `TripleBool` 枚举类型.
如果在每个文件中都定义一遍显然不是什么好办法.
我们可以使用导入语句将 其他文件中的类型导入到当前文件中.
例如, 我们将 `TripleBool` 存放在 `common.proto` 文件中.
在需要的地方 ,使用  `import` 关键字导入即可.
例如, 我们可以这样编写`response.proto`

``` protobuf
syntax = "proto3";
import "common.proto"

message BasicResponse{
    TripleBool succeed = 1; // 使用时, 像普通类型使用即可.
    string msg = 2;
}
```

默认情况下, 如果`common.proto` 文件自身还 `import` 了另一个文件 `other.proto`, 你是不能直接使用`other.proto`中定义的类型的.
除非 `common.proto` 在 `import` `other.proto` 时将其标记为 `public`.
例如:

```  protobuf
// common.proto
syntax = "proto3";
import public "other.proto"
import "private.proto"
// ... 
```

```  protobuf
// response.proto
syntax = "proto3";
import "common.proto"
// 可以在此处使用 "other.proto" 中的类型.
// 不过 "private.proto" 中的类型依然不能使用.
```

### 引用文件路径的查找方式

需要注意的是, 在查找引用的文件时, 查找的起点并非不是当前文件. 
起点可以在编译是通过指定参数 `-I` 或是 `--proto_path` 指定.  如果不指定, 则以当前工作目录为起点.
通常为了减少路径的频繁修改, 我们通常将 `proto_path` 指定为项目的根目录.
不过并不建议使用内联/嵌套的方式定义命名空间, 我们可以直接使用包.

## 类型的嵌套/内联

除了上面说到的枚举之外, 消息结构体也可以内联.
如:

``` protobuf
message AAA {
    message AA {
        message A {
            bool v = 1;// 你可以嵌套任意多层
        }
    }
}
message B {
    AAA.AA.A a = 1; // 嵌套类型使用 . 依次分隔每一层.
}
```

## 包, 命名空间

我们常常需要使用不同的包来区分不同的业务场景.
只需要 proto 文件中增加一行 `package` 指令, 就可以了创建一个包了.
例如

``` protobuf
// error.proto
syntax = "proto3";
package base.common
message Error {
    int32 err_code = 1;
    string message = 2;
}
```

``` protobuf
// response.proto
syntax = "proto3";
import "src/base/common/error.proto"
message Reponse {
    bool success = 1;
    base.common.Error err = 2;
}
```

在不同语言上, 会生成不同的代码用来表示包.
例如 使用 `java` 则会生成一个个包, 不过你可以通过指定  `java_package` 来单独指定为其他名字也可以.
`golang` 也是类似的, 同样可以通过指定  `go_package` 来单独指定为其他名字
`python` 就直接忽略了, python 没有包的概念 :).
其他语言[参考这里](https://developers.google.com/protocol-buffers/docs/proto3#packages)
当发生类型名称冲突时, 依赖就近原则.

## 更新消息结构的注意事项

当已有的消息定义不能满足需要时, 你可能会需要对字段有一定增减, 修改时需要注意以下事项:

* 不要修改现有字段的id.
* 对于新增的字段, 将旧的数据解析时为新版的消息结构时, 并不会报错, 缺少的字段会使用 0 值填充. 注意, 你并不能区分是缺少字段还是字段故意设定为0值.
* 对于新增的字段, 使用旧的消息结构解析心得数据时, 新增的字段会被自动忽略. 目前对于版本3 来说, 这些数据就永远丢失了. 重新序列化时就不复存在了. 官方文档说可能会在 3.5 中将忽略字段保留起来, 可以看[这里](https://developers.google.com/protocol-buffers/docs/proto3#unknowns)
* 如果你想要删除字段, 请使用 `废弃字段` 需要丢弃的字段标记起来. 具体看下面的具体介绍
* 某些数据直接互相兼容, 可以在不增减的情况直接修改, 具体可以[参考这里](https://developers.google.com/protocol-buffers/docs/proto3#updating)
  
### 废弃字段

当你想要删除字段时, 需要特别注意. 
如果直接删除, 那么可能会导致不同版本的客户端和服务器相连接时, 数据结构不一致导致出现莫名其妙的问题.
这个时候, 你可能需要显示声明废弃字段, 为他们保留位置.

``` protobuf
message SomeMessage {
    reserved 2, 3, 10;
    reserved "field_a", "field_b";
}
```

注意, 这两种声明方式不能同时写在一行.

## 定义一个服务

proto 中除了可以定义消息结构之外, 还可以定义rpc 接口.
例如, 我们可以定义一个登陆接口

``` protobuf
service UserService {
    rpc Login(LoginRequest) return (LoginResponse)
}
```

## 编译生成语言相关的代码

要编译protobuf 为语言相关的代码, 你需要首先安装编译器.
编译器的作用就是使用特定的语言将你写的定义文件实现出来.
比如你定义了一个message:

``` protobuf
message Book {
    string name = 1;
    float price = 2;
}
```

将会编译为c++代码(节选)

``` c++
class Book : public ::PROTOBUF_NAMESPACE_ID::Message {
    public:
        void set_name(const ::std::string& value);
        void set_price(float value);
    // 剩余省略
};
```

然后你在c++ 代码中就可以使用并序列化数据了.
例如:

``` c++
Book book;
book.set_price(12.0);
book.set_name("hello, world");
fstream output;
book.SerializeToOstream(&output);
```

编译的第一步肯定就是要先安装编译器了, 你可以从[这里下载](https://github.com/protocolbuffers/protobuf/releases)
或者自己从[源码编译](https://github.com/protocolbuffers/protobuf/blob/master/src/README.md)
或者如果你使用mac 可以直接使用 [homebrew](https://brew.sh/) 安装 `brew install protobuf`
安装好之后, 就可以使用 `protoc` 命令编译了
基础版的命令是这样的 `protoc --proto_path=IMPORT_PATH --xxx_out=DST_DIR path/to/file.proto`
`--proto_path=IMPORT_PATH` 是在使用 `import` 命令时所使用的路径, 一般设置为项目的根目录. 
`--proto_path` 可以缩写为 `-I`
`xxx_out` 表示要输出的语言, 后面 `DST_DIR` 是输出的目的文件夹路径, 例如 `cpp_out`, 注意, 目标文件夹需要存在.
`path/to/file.proto` 则是你的 `protobuf` 定义文件, 路径以 `IMPORT_PATH` 开始查找.

### 关于golang 

想要编译为golang 语言的同学请注意, 需要先安装golang 的扩展支持
`go get -u github.com/golang/protobuf/protoc-gen-go`
`--go_out` 这个编译参数除了路径之外, 还支持其他参数, 例如`plugins`, `import_path`, 完整列表[参考这里](https://github.com/golang/protobuf/#parameters)
例如我们需要使用 [grpc](https://grpc.io/) 框架的支持.
可以这样使用

``` bash
protoc -I helloworld/ helloworld/helloworld.proto --go_out=plugins=grpc:helloworld
```
