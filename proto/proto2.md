
# 第二章.proto2指南

本指南介绍如何使用协议缓冲区语言来构建协议缓冲区数据，包括.proto文件语法以及如何从.proto文件生成数据访问类。 它涵盖协议缓冲区语言的proto2版本：有关较新的proto3语法的信息，请参阅Proto3语言指南。

对于使用本文档中描述的许多功能的逐步示例，请参阅所选语言的教程。

## 第一节.定义一个消息类型

首先，让我们来看一个非常简单的例子。让我们来看你想定义一个搜索数据消息格式，每个搜索请求有一个字符串，将获得一个你感兴趣的结果，并且对结果做了分页处理，你使用下面这个.proto文件定义消息类型。

    message SearchRequest {
      required string query = 1;
      optional int32 page_number = 2;
      optional int32 result_per_page = 3;
    }

SearchRequest消息定义了三个字段（名字/值对），数据中的没有一块你都要包含在这个message类型中，每个字段有一个名字和一个类型

### 一.指定字段类型

在上面的例子中，所有字段都是标量类型：两个整形（page_number和result_per_page）和一个字符串类型(query)。当然，你也可以指定一个字段为复合类型，包括枚举和其他的消息类型

### 二.分配标签

如你看到的一样，每个字段在消息定义中都有一个唯一的数字标志。这些标志在消息二进制格式中用来标识你的字段，只要你的消息类型在使用这些标识值都不会改变。


As you can see, each field in the message definition has a unique numbered tag. These tags are used to identify your fields in the message binary format, and should not be changed once your message type is in use. Note that tags with values in the range 1 through 15 take one byte to encode, including the identifying number and the field's type (you can find out more about this in Protocol Buffer Encoding). Tags in the range 16 through 2047 take two bytes. So you should reserve the tags 1 through 15 for very frequently occurring message elements. Remember to leave some room for frequently occurring elements that might be added in the future.








