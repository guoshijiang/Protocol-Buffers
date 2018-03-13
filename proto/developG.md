
# 第一章：概述

protoBuf一种语言中立，平台无关，可扩展的串行化结构化数据的方式，用于通信协议，数据存储等。本文档针对希望在其应用程序中使用协议缓冲区的Java，C++或Python开发人员。 本概述介绍了协议缓冲区，并告诉您需要做什么才能开始，然后你可以继续学习教程或深入研究协议缓冲区源码。 还讲述了如何编写.proto文件的语言和样式。

## 第一节.Protocol Buffer简介

### 一.什么是Protocol Buffer

协议缓冲区能够灵活，高效的自动化地序列化结构化数据的，和XML、Json比较类似，但它更小，更快，更简单。 您可以定义一次数据的结构，然后您可以使用特殊的生成源代码轻松地将结构化数据写入各种数据流和从各种数据流读取并使用各种语言。 您甚至可以更新您的数据结构，而不会中断根据“旧”格式编译的已部署程序。

### 二.Protocol Buffer是怎么样工作的

您可以通过在.proto文件中定义协议缓冲区消息类型来指定您想要序列化数据结构。每个协议缓冲区消息是一个小的逻辑信息记录，包含一系列名称-值对。 下面是.proto文件的一个非常基本的例子，该文件定义了一条包含有关人员信息的消息：

    message Person {
      required string name = 1;
      required int32 id = 2;
      optional string email = 3;

      enum PhoneType {
        MOBILE = 0;
        HOME = 1;
        WORK = 2;
      }

      message PhoneNumber {
        required string number = 1;
        optional PhoneType type = 2 [default = HOME];
      }

      repeated PhoneNumber phone = 4;
    }
    
就像你看到的一样，消息的格式非常简单，每个消息有一个或者多个独特的数字域，并且每个域一个name和一个value类型,value类型可以是数字（正兴或者浮点型），booleans型，strings型，raw byte类型，甚至是上面例子中没有提到的其他protocol buffer消息类型，允许你结构化数据层级。你可以指定可选字段，必须字段，重复字段。在Protocol Buffer语言指南中，你可以找到更多关于怎么去编写.proto文件的方式。

一旦你已经定义了你的消息，您可以在.proto文件上为应用程序的语言运行协议缓冲区编译器来生成数据访问类。这将为每个字段提供了简单的访问器（如name（）和set_name（））以及将整个结构序列化到/从原始字节解析的方法-例如，如果您选择的语言是C++，则运行编译器上面的例子将生成一个名为Person的类。 然后，您可以在您的应用程序中使用此类来填充，序列化并检索Person协议缓冲区消息。 你可能会写这样的代码

    Person person;
    person.set_name("John Doe");
    person.set_id(1234);
    person.set_email("jdoe@example.com");
    fstream output("myfile", ios::out | ios::binary);
    person.SerializeToOstream(&output);
    
然后，你可以将消息读回

    fstream input("myfile", ios::in | ios::binary);
    Person person;
    person.ParseFromIstream(&input);
    cout << "Name: " << person.name() << endl;
    cout << "E-mail: " << person.email() << endl;


您可以将新字段添加到消息格式中，而不会破坏向后兼容性; 解析时旧的二进制文件简单地忽略新字段。 所以如果你有一个使用协议缓冲区作为数据格式的通信协议，你可以扩展你的协议，而不用担心破坏现有的代码。

你将会在API关联部分找到使用Protocol Buffer生成代码的链接，并且你将会找出更多关于在协议缓冲区编码中如何编码协议缓冲区消息

### 三.为什么不仅仅使用XML

和XML相比较，Protocol buffers在序列化结构数据上面比XML会有更多的优势：

1.Protocol buffers更简单
2.Protocol buffers更小
3.Protocol buffers更快
4.Protocol buffers耦合性更少
5.生成更易于以编程方式使用的数据访问类

例如：在XML中，定义一个带有姓名和邮箱的人的数据模型，你需要这样做：

    <person>
        <name>John Doe</name>
        <email>jdoe@example.com</email>
    </person>
    
使用protocol buffer数据格式

    person {
      name: "John Doe"
      email: "jdoe@example.com"
    }

当这个消息被编码为协议缓冲区二进制格式（上面的文本格式只是一个便于用户进行调试和编辑的人类可读表示）时，它可能是28个字节长，需要大约100-200纳秒才能解析。 如果删除空格，XML版本至少为69个字节，解析需要大约5,000-10,000纳秒。

另外，操作协议缓冲区要容易得多：

    cout << "Name: " << person.name() << endl;
    cout << "E-mail: " << person.email() << endl;
    
而XML格式的数据需要像下面这样做

    cout << "Name: "
           << person.getElementsByTagName("name")->item(0)->innerText()
           << endl;
    cout << "E-mail: "
           << person.getElementsByTagName("email")->item(0)->innerText()
           << endl;

但是，协议缓冲区并不总是比XML更好的解决方案-例如，协议缓冲区不是用标记（例如HTML）对基于文本的文档建模的好方法，因为您无法轻松地将结构与文本交错。 另外，XML是人类可读的，可编辑的， 协议缓冲区至少以其本地格式不是。 XML在某种程度上也是自我描述的。 如果您有消息定义（.proto文件），协议缓冲区才有意义。

### 四.Protocol Buffers的使用

下载包-完整的源码包中包含了Java,Python和C++的Protocol Buffer编译器，也包含你需要的IO测试。创建和安装你的编译器，在README文件中有详细的说明，在后面的内容中将介绍。

### 五.proto3 介绍

我们最新的版本3版本引入了一种新的语言版本-Protocol Buffers语言版本3（又名proto3），以及我们现有语言版本（又名proto2）中的一些新功能。Proto3简化了协议缓冲区语言，既易于使用又可用于更广泛的编程语言：我们当前的版本可让您以Java，C++，Python，Java Lite，Ruby，JavaScript，Objective-C和C＃。 另外，您可以使用golang/protobuf Github存储库中提供的最新Go protoc插件为Go生成proto3代码。更多语言正在筹备中。

请注意，两种语言版本API不完全兼容。 为了避免给现有用户带来不便，我们将继续在新的协议缓冲版本中支持以前的语言版本。

你能在发行说明中看到与当前默认版本的主要差异，并了解Proto3语言指南中的proto3语法。 proto3的完整文档即将推出！

（如果proto2和proto3的名字看起来有点混乱，那是因为最初开源的协议缓冲区时，它实际上是Google的第二个语言版本-也被称为proto2，这也是开源版本号从v2.0.0开始的原因）。

### 六.历史简介

协议缓冲区最初是在Google开发的，用于处理索引服务器请求/响应协议。 在协议缓冲区之前，有一种请求和响应格式，用于手动编组/解组请求和响应，并支持多种版本的协议。 这导致了一些非常难看的代码，例如：

    if (version == 3) {
       ...
     } else if (version > 4) {
       if (version == 5) {
         ...
       }
       ...
     }
     
明确格式化的协议也使新协议版本的推出变得复杂，因为开发者必须确保请求发起者与处理请求的实际服务器之间的所有服务器都能理解新协议，然后才能触发交换机开始使用新协议。

Protocol buffers的设计开发的确解决了很多问题

* 可以很容易地引入新的字段，并且不需要检查数据的中间服务器可以简单地解析并传递数据，而无需了解所有字段。
* 格式更加自我描述，可以用各种语言来处理（C ++，Java等）

然而，用户仍然需要手写他们自己解析的代码

随着系统的发展，它获得了许多其他功能和用途：

* 自动生成的序列化和反序列化代码避免了手动解析的需要。
* 除了用于短期RPC（远程过程调用）请求之外，人们开始将协议缓冲区用作持久存储数据的便捷自描述格式（例如，在Bigtable中）。
* 服务器RPC接口开始被声明为协议文件的一部分，协议编译器生成存根类，用户可以使用服务器接口的实际实现来覆盖它们。

协议缓冲区现在是Google用于数据的通用语言-在撰写本文时，在12,183个.proto文件中Google代码树中定义了48,162种不同的消息类型。 它们既用于RPC系统，也用于在各种存储系统中持久存储数据。






