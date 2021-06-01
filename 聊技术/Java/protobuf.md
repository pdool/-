​		![](../../img/20200829/1.png)

​		游戏服务器和客户端的通信有很多种形式，有的用http，有的用websocket，不过最常见的还是socket服务器，socket 服务器在游戏中是最常见的，至于为什么和怎么创建，等以后再说，今天先来聊聊服务器和客户端交谈的协议。协议的定义是服务端和客户端沟通的结果，形成一致的数据格式，这样大家才好解析，知道对方在说什么，在做什么。

​	在最初的时候有的人自定义格式，虽然紧凑，但是可能会存在一些问题，不够稳定。也有的人用xml，也有人用json，存在的问题就是格式虽然很好，但是网络包太大，不太适合，问题存在必然就要解决，有没有一种方案可以解决上面的问题？答案显而易见，就是今天聊的protobuf。

​	protobuf 是谷歌开源的跨平台的一种通讯协议，更紧凑，更高效。废话不多说，进入正文。

## 1、Java项目引用

pom.xml 中加入以下依赖，版本可以自己根据需要进行选择

```
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.6.1</version>
</dependency>
```



### 2、protobuf 的文件定义格式

```protobuf

option java_package ="com.gamwatcher.soulmsg";
option java_outer_classname = "SoulMsg";
option java_multiple_files = true;
message SOUL_UP_OUT{
    required int64 uid =1;
    repeated int64 costuid =2;
    optional int64 useExp = 3;
}
```

### 基础类型

| .proto类型 | java类型   | C++类型 | 备注                                                         |
| ---------- | ---------- | ------- | ------------------------------------------------------------ |
| double     | double     | double  |                                                              |
| float      | float      | float   |                                                              |
| int32      | int        | int32   | 使用可变长编码方式。编码负数时不够高效——如果你的字段可能含有负数，那么请使用sint32。 |
| int64      | long       | int64   | 使用可变长编码方式。编码负数时不够高效——如果你的字段可能含有负数，那么请使用sint64。 |
| unit32     | int[1]     | unit32  | 总是4个字节。如果数值总是比总是比228大的话，这个类型会比uint32高效。 |
| unit64     | long[1]    | unit64  | 总是8个字节。如果数值总是比总是比256大的话，这个类型会比uint64高效。 |
| sint32     | int        | int32   | 使用可变长编码方式。有符号的整型值。编码时比通常的int32高效。 |
| sint64     | long       | int64   | 使用可变长编码方式。有符号的整型值。编码时比通常的int64高效。 |
| fixed32    | int[1]     | unit32  |                                                              |
| fixed64    | long[1]    | unit64  | 总是8个字节。如果数值总是比总是比256大的话，这个类型会比uint64高效。 |
| sfixed32   | int        | int32   | 总是4个字节。                                                |
| sfixed64   | long       | int64   | 总是8个字节。                                                |
| bool       | boolean    | bool    |                                                              |
| string     | String     | string  | 一个字符串必须是UTF-8编码或者7-bit ASCII编码的文本。         |
| bytes      | ByteString | string  | 可能包含任意顺序的字节数据                                   |

### 特殊字段

| 英文     | 中文                                                | 备注                                           |
| -------- | --------------------------------------------------- | ---------------------------------------------- |
| enum     | 枚举(数字从零开始) 作用是为字段指定某”预定义值序列” | enum Type {MAN = 0;WOMAN = 1; OTHER= 3;}       |
| message  | 消息体                                              | message User{}                                 |
| repeated | 数组/集合                                           | repeated User users  = 1                       |
| import   | 导入定义                                            | import "protos/other_protos.proto"             |
| //       | 注释                                                | //用于注释                                     |
| extend   | 扩展                                                | extend User {}                                 |
| package  | 包名                                                | 相当于命名空间，用来防止不同消息类型的明明冲突 |

## 2、生成java类

​	下载protoc： https://github.com/protocolbuffers/protobuf/releases 

```
protoc.exe --java_out = ../../src/main/java **.proto
```

## 3、使用协议

```
SOUL_UP_OUT.Builder builder = SOUL_UP_OUT.newBuilder();
builder.setUid(1);
builder.addAllCostUid(costUidList);
builder.setUserExp(1000)
builder.build()
```

### 4、如何在游戏项目中使用

正常的协议格式:

len + 加密的 [headMsgId + proto二进制数据]

常用的加密算法：md5和rsa，DES，选择一个简单的效率高的，如果游戏大火了可以换一个稍微复杂的加密算法，小事情，不重要

客户端解析出根据长度读出数据长度进行解析。so easy!!!，服务端同样的规则。客户端和服务器通信就是这么简单。

总结： protobuf 不过是一个协议格式，省去了我们自定义消息的过程，既然有现成的轮子就没必要自己造了，况且我们造的还不如别人，先会用，再去了解原理，没什么大不了。

![](../../img/20200829/2.png)