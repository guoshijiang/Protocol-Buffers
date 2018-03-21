
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

如你看到的一样，每个字段在消息定义中都有一个唯一的数字标志。这些标志在消息二进制格式中用来标识你的字段，只要你的消息类型在使用这些标识值都不会改变。请注意，值为1到15的变量需要一个字节进行编码，包括标识号和字段类型（您可以在协议缓冲区编码中找到更多相关信息）。 范围16到2047中的标签需要两个字节。 因此，您应该为非常频繁出现的消息元素预留标签1至15。 请记住为将来可能添加的频繁出现的元素留出一些空间。

最小的标识数字可以指定为1，最大值是2的29次方-1，或者是536,870,911。你不能使用数字19000到19999(FieldDescriptor::kFirstReservedNumber到FieldDescriptor::kLastReservedNumber), 因为它们是为协议缓冲区实现保留的。如果您在.proto中使用这些保留的数字之一，则协议缓冲区编译器会发出错误。 同样，您不能使用任何以前保留的标签。

### 三.指定字段的规则

您指定消息字段是以下之一：

`required`：格式正确的消息必须具有该字段中的一个。
`optional`：格式正确的消息可以有零个或一个此字段（但不能超过一个）。
`repeated`：该字段可以在格式良好的消息中重复任意次数（包括零）。 重复值的顺序将被保留。

由于历史原因，标量数字类型的重复字段因为它们可能是未被编码为已启用。 新代码应该使用特殊选项[packed = true]来获得更高效的编码。 例如：

    repeated int32 samples = 4 [packed=true];

您可以在协议缓冲区编码中找到更多关于打包编码的信息。

### 四.添加更多的消息类型

多个消息类型可以在一个.proto文件中定义。 如果您定义了多个相关消息，这非常有用-例如，如果您想要定义与SearchResponse消息类型相对应的答复消息格式，则可以将其添加到相同的.proto文件中：

    message SearchRequest {
      required string query = 1;
      optional int32 page_number = 2;
      optional int32 result_per_page = 3;
    }

    message SearchResponse {
     ...
    }

注意：

组合消息导致膨胀虽然可以在单个.proto文件中定义多个消息类型（如消息，枚举和服务），但是当在单个文件中定义了具有不同依赖性的大量消息时，它也可能导致依赖性膨胀。 建议尽可能在每个.proto文件中包含尽可能少的消息类型。

### 五.添加注释

要向.proto文件添加注释，请使用C / C ++样式//和/ * ... * /语法。

    /* SearchRequest represents a search query, with pagination options to
     * indicate which results to include in the response. */

    message SearchRequest {
      required string query = 1;
      optional int32 page_number = 2;  // Which page number do we want?
      optional int32 result_per_page = 3;  // Number of results to return per page.
    }

### 六.保留字段

如果您通过完全删除某个字段或将其注释掉来更新消息类型，那么未来的用户可以在对该类型进行自己的更新时重新使用该标签号。 如果稍后加载相同的.proto的旧版本，包括数据损坏，隐私错误等，则会导致严重问题。 确保这种情况不会发生的一种方法是指定已删除字段的字段标记（和/或名称，这也可能会导致JSON序列化问题）被保留。 如果将来的任何用户试图使用这些字段标识符，协议缓冲区编译器将会投诉。

    message Foo {
      reserved 2, 15, 9 to 11;
      reserved "foo", "bar";
    }
    
 请注意，您不能在同一保留语句中混合字段名称和标签号码。
 
 ### 七.你的.proto产生了什么？
 
 当您在.proto上运行协议缓冲区编译器时，编译器会以您选择的语言生成代码，您需要使用文件中描述的消息类型，包括获取和设置字段值，将消息序列化为 输出流，并从输入流中解析消息。
 
* 对于C ++，编译器会从每个.proto生成一个.h和.cc文件，并为您的文件中描述的每种消息类型生成一个类。
* 对于Java，编译器会为每个消息类型生成一个带有类的.java文件，以及用于创建消息类实例的特殊Builder类。
* Python有一点不同-Python编译器在.proto中生成一个带有每种消息类型的静态描述符的模块，然后与元类一起使用，以在运行时创建必要的Python数据访问类。
* 对于Go，编译器会为文件中的每种消息类型生成一个.pb.go文件。

您可以按照所选语言的教程了解更多有关使用每种语言的API的信息。 有关更多API详细信息，请参阅API部分。API部分将在后面文章中讲解

## 第二节.标量值类型

标量消息字段可以具有以下类型之一;该表显示.proto文件中指定的类型以及自动生成的类中的相应类型：

 |  .proto 类型  |    C++类型   |     Java类型      |     python类型        |      Go类型      | 
 |--------------|--------------|------------------|----------------------|------------------|  
 |   double		|   double	   |      double	  |       float	         |     *float64     |
 |   float		|   float	   |      float	      |       float          |     *float32     |
 |   int32	    |	int32	   |      int	      |       int	         |     *int32       |
 |   int64		|   int64	   |      long	      |      int/long[3]	 |     *int64       |
 |   uint32		|   uint32	   |      int[1]	  |      int/long[3]	 |     *uint32      |
 |   uint64		|   uint64	   |      long[1]	  |      int/long[3]	 |     *uint64      |
 |   sint32	  	|   int32	   |      int	      |      int	         |     *int32       |
 |   sint64		|   int64	   |      long	      |      int/long[3]	 |     *int64       |
 |   fixed32	|   uint32	   |      int[1]	  |      int	         |     *uint32      |
 |   fixed64	|   uint64	   |      long[1]	  |      int/long[3]	 |     *uint64      |
 |   sfixed32	|	int32	   |      int	      |      int	         |     *int32       |
 |   sfixed64	|	int64	   |      long	      |      int/long[3]	 |     *int64       |
 |   bool		|   bool	   |      boolean	  |      bool	         |     *bool        |
 |   string		|   string	   |      String	  |      str/unicode[4]	 |     *string      |
 |   bytes      |	string	   |      ByteString  |      str             |     []byte       |


## 第三节.可选择字段和默认值

就像上面提到的，消息中的元素标签是可选的。一个完美格式的消息可能不会包含一个可选择的标签。一个消息被解析时，如果消息中不包含一个可选的元素，在对象解析中对应的域会为相应的字段设置默认值。默认值会被指定为消息描述的一部分。例如：我们希望给SearchRequest的result_per_page值提供一个默认值为10

    optional int32 result_per_page = 3 [default = 10];

如果默认值没有被指定为一个可选择元素，将用类型指定默认值来替代：对于strings,默认值是一个空字符串，对于bool型，默认值是false，对于数字类型，默认值是0.对于枚举，默认值是枚举定义列表中的第一个值。这将意味着定义枚举时你需要给枚举列表中的第一个变量添加一个值。

## 第四节.枚举

当你定义一个消息类型时，你可能想这些字段中的一个字段的值在预定义列表中仅仅有一个。例如：你想要为每一个SearchRequest添加一个全集字段，这个全集字段可能是UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS或者VIDEO.这是非常简单的在消息定义中通过添加一个枚举-枚举类型的字段仅仅有一个值指定设置的常量做为其值（如果你尝试提供不同的值，解释器将会将其看成一个未知字段）。在下面的例子中，我们添加了一个带有所有可能的值得枚举集合，字段类型是集合：

    message SearchRequest {
      required string query = 1;
      optional int32 page_number = 2;
      optional int32 result_per_page = 3 [default = 10];
      enum Corpus {
        UNIVERSAL = 0;
        WEB = 1;
        IMAGES = 2;
        LOCAL = 3;
        NEWS = 4;
        PRODUCTS = 5;
        VIDEO = 6;
      }
      optional Corpus corpus = 4 [default = UNIVERSAL];
    }

假设相同的值对于不同的枚举常量你可以起别名。做这件事需要设置allow_alias option为true，如果当别名被发现，其他的protocol编译器将产生一个错误的消息

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

枚举器常量必须在32-位整形的范围内。由于枚举值在线路上使用varint编码，所以负值效率不高，因此不推荐使用。你可在消息内部定义枚举，就像上面的列子一样，或者在外部-在proto文件中你的定义中这枚举可以被重用。您还可以使用语法MessageType.EnumType将一个消息中声明的枚举类型用作不同消息中字段的类型。

当你对一个使用了枚举的.proto代码运行rotocol buffer编译器，产生的代码将会对应java或者C++中的枚举，对于Python来说会产生一个特别的EnumDescriptor类，这个类用来在运行产生类设置一些整形的标志常量

有关如何在应用程序中使用消息枚举的更多信息，请参阅所选语言的生成代码部分内容。

### 一.保留值

如果你通过完全移除一个枚举实体来更新一个枚举类型，或者将其注释掉，将来的用户更新他们自己的类型时能够重新使用这个数字值。如果以后加载相同的旧版本的包括数据出错，自身缺陷等的.proto文件这将会导致服务器问题。一个有效的方法确保这种情况不会发生把要删除的实体的数字值保留。如果将来的用户去使用这些标识码时protocol buffer编译器将会去拒绝。你可以指定你的保留数字值范围到最大可能的数和最大的关键字。

    enum Foo {
      reserved 2, 15, 9 to 11, 40 to max;
      reserved "FOO", "BAR";
    }

注意：在同一个保留声明里面不能混合字段名字和数字值

## 第五节.使用其他的消息类型

你可以使用其他的消息作为字段类型。例如，你想在每一个SearchResponse消息中包含Result消息，像这样做，你可以在同一个.proto中定义一个Result消息类型，然后在SearchResponse中的Result中指定一个字段类型：

    message SearchResponse {
      repeated Result results = 1;
    }

    message Result {
      string url = 1;
      string title = 2;
      repeated string snippets = 3;
    }

### 一.引入定义

在上面的例子中，`Result`消息类型在与`SearchResponse`相同的文件中定义-如果您希望用作字段类型的消息类型已经在另一个`.proto`文件中定义，该怎么办？

你可以通过在其他`proto`文件定义然后通过import关键字引入他们，引入另一个proto的定义，在文件顶部引入import陈述：

    import "myproject/other_protos.proto";

默认情况下，你仅仅只能使用定义直接从`.proto`文件中引入。然而，有时你可能需要将`.proto`文件移动到另一个新的位置。

默认情况下，您只能使用直接导入的.proto文件中的定义。 但是，有时您可能需要将.proto文件移至新位置。 不是直接移动.proto文件，而是在一次更改中更新所有调用站点，现在您可以在旧位置放置一个虚拟.proto文件，以使用导入公共概念将所有导入转移到新位置。 任何人输入包含导入公开声明的协议都可以传递依赖进口公共依赖关系。 例如：

    // new.proto
    // 所有的定义被移动到这儿

    // old.proto
    // 这个proto所有的客户端都能引入
    import public "new.proto";
    import "other.proto";

    // client.proto
    import "old.proto";
    // 你可以从old.proto和new.proto使用定义,但不能是other.proto



The protocol compiler searches for imported files in a set of directories specified on the protocol compiler command line using the -I/--proto_path flag. If no flag was given, it looks in the directory in which the compiler was invoked. In general you should set the --proto_path flag to the root of your project and use fully qualified names for all imports.


