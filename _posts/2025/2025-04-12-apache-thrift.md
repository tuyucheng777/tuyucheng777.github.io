---
layout: post
title:  使用Apache Thrift
category: apache
copyright: apache
excerpt: Apache Thrift
---

## 1. 概述

在本文中，我们将探索如何借助名为[Apache Thrift](https://thrift.apache.org/)的RPC框架开发跨平台客户端-服务器应用程序。

我们将介绍：

- 使用IDL定义数据类型和服务接口
- 安装库并生成不同语言的源代码
- 用特定语言实现定义的接口
- 实现客户端/服务器软件

如果你想直接查看示例，请直接进入第5节。

## 2. Apache Thrift

Apache Thrift最初由Facebook开发团队开发，目前由Apache维护。

与管理跨平台对象序列化/反序列化过程的[Protocol Buffer](https://www.baeldung.com/spring-rest-api-with-protocol-buffers)相比，**Thrift主要关注系统组件之间的通信层**。

Thrift使用特殊的接口描述语言(IDL)来定义数据类型和服务接口，它们存储为.thrift文件，稍后由编译器用作输入，以生成通过不同编程语言进行通信的客户端和服务器软件的源代码。

要在项目中使用Apache Thrift，请添加以下Maven依赖：

```xml
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.10.0</version>
</dependency>
```

可以在[Maven仓库](https://mvnrepository.com/artifact/org.apache.thrift/libthrift)中找到最新版本。

## 3. 接口描述语言

如前所述，[IDL](https://thrift.apache.org/docs/idl)允许使用中性语言定义通信接口，以下列出了当前支持的类型。

### 3.1 基本类型

- bool：布尔值(true或false)
- 字节：8位有符号整数
- i16：16位有符号整数
- i32：32位有符号整数
- i64：64位有符号整数
- double：64位浮点数
- string：使用UTF-8编码的文本字符串

### 3.2 特殊类型

- binary：未编码字节序列
- optional：Java 8的Optional类型

### 3.3 结构体

Thrift的structs相当于面向对象编程(OOP)语言中的类，但没有继承。结构体包含一组强类型字段，每个字段都有一个唯一的名称作为标识符，字段可以有各种注解(数字字段ID、可选默认值等)。

### 3.4 容器

Thrift容器是强类型容器：

- list：元素的有序列表
- set：一组无序的唯一元素
- map<type1,type2\>：严格唯一键到值的映射

容器元素可以是任何有效的Thrift类型。

### 3.5 异常

异常在功能上等同于structs，只是它们继承自本机异常。

### 3.6 服务

服务实际上是使用Thrift类型定义的通信接口，它们由一组命名函数组成，每个函数都有一个参数列表和一个返回类型。

## 4. 源代码生成

### 4.1 语言支持

目前支持的语言有很多：

- C++
- C#
- Go
- Haskell
- Java
- Javascript
- Node.js
- Perl
- PHP
- Python
- Ruby

你可以在[此处](https://thrift.apache.org/lib/)查看完整列表。

### 4.2 使用库的可执行文件

只需下载[最新版本](https://thrift.apache.org/download)，如有必要，构建并安装它，然后使用以下语法：

```shell
cd path/to/thrift
thrift -r --gen [LANGUAGE] [FILENAME]
```

在上面设置的命令中，[LANGUAGE\]是受支持的语言之一，[FILENAME\]是一个具有IDL定义的文件。

注意-r标志，它告诉Thrift一旦注意到给定.thrift文件中包含内容，就递归生成代码。

### 4.3 使用Maven插件

在你的pom.xml文件中添加插件：

```xml
<plugin>
    <groupId>org.apache.thrift.tools</groupId>
    <artifactId>maven-thrift-plugin</artifactId>
    <version>0.1.11</version>
    <configuration>
        <thriftExecutable>path/to/thrift</thriftExecutable>
    </configuration>
    <executions>
        <execution>
            <id>thrift-sources</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

之后只需执行以下命令：

```shell
mvn clean install
```

请注意，此插件将不再维护，请访问[此页面](https://github.com/dtrott/maven-thrift-plugin)了解更多信息。

## 5. 客户端-服务器应用程序示例

### 5.1 定义Thrift文件

让我们编写一些带有异常和结构的简单服务：

```text
namespace cpp cn.tuyucheng.taketoday.thrift.impl
namespace java cn.tuyucheng.taketoday.thrift.impl

exception InvalidOperationException {
    1: i32 code,
    2: string description
}

struct CrossPlatformResource {
    1: i32 id,
    2: string name,
    3: optional string salutation
}

service CrossPlatformService {

    CrossPlatformResource get(1:i32 id) throws (1:InvalidOperationException e),

    void save(1:CrossPlatformResource resource) throws (1:InvalidOperationException e),

    list <CrossPlatformResource> getList() throws (1:InvalidOperationException e),

    bool ping() throws (1:InvalidOperationException e)
}
```

如你所见，语法非常简单，一目了然。我们定义了一组命名空间(针对每种实现语言)、一个异常类型、一个结构体，以及最终一个将在不同组件之间共享的服务接口。

然后将其存储为service.thrift文件。

### 5.2 编译并生成代码

现在是时候运行编译器来为我们生成代码了：

```shell
thrift -r -out generated --gen java /path/to/service.thrift
```

如你所见，我们添加了一个特殊标志-out来指定生成文件的输出目录。如果你没有收到任何错误，则生成的目录将包含3个文件：

- CrossPlatformResource.java
- CrossPlatformService.java
- InvalidOperationException.java

让我们通过运行以下命令生成该服务的C++版本：

```shell
thrift -r -out generated --gen cpp /path/to/service.thrift
```

现在我们获得了同一服务接口的2个不同的有效实现(Java和C++)。

### 5.3 添加服务实现

虽然Thrift已经为我们完成了大部分工作，但我们仍然需要自己编写CrossPlatformService的实现。为此，我们只需要实现一个CrossPlatformService.Iface接口：

```java
public class CrossPlatformServiceImpl implements CrossPlatformService.Iface {

    @Override
    public CrossPlatformResource get(int id) throws InvalidOperationException, TException {
        return new CrossPlatformResource();
    }

    @Override
    public void save(CrossPlatformResource resource) throws InvalidOperationException, TException {
        saveResource();
    }

    @Override
    public List<CrossPlatformResource> getList() throws InvalidOperationException, TException {
        return Collections.emptyList();
    }

    @Override
    public boolean ping() throws InvalidOperationException, TException {
        return true;
    }
}
```

### 5.4 编写服务器

正如我们所说，我们想要构建一个跨平台的客户端-服务器应用程序，所以我们需要一个服务器。Apache Thrift的优点在于它有自己的客户端-服务器通信框架，这使得通信变得轻而易举：

```java
public class CrossPlatformServiceServer {
    public void start() throws TTransportException {
        TServerTransport serverTransport = new TServerSocket(9090);
        server = new TSimpleServer(new TServer.Args(serverTransport)
                .processor(new CrossPlatformService.Processor<>(new CrossPlatformServiceImpl())));

        System.out.print("Starting the server... ");

        server.serve();

        System.out.println("done.");
    }

    public void stop() {
        if (server != null && server.isServing()) {
            System.out.print("Stopping the server... ");

            server.stop();

            System.out.println("done.");
        }
    }
}
```

首先，我们需要定义一个传输层，并实现TServerTransport接口(或者更准确地说是抽象类)。由于我们讨论的是服务器，因此需要提供一个监听端口。然后，我们需要定义一个TServer实例，并从以下可用实现中选择一个：

-TSimpleServer：用于简单服务器
-TThreadPoolServer：用于多线程服务器
-TNonblockingServer：用于非阻塞多线程服务器

最后，为所选服务器提供已由Thrift为我们生成的处理器实现，即CrossPlatofformService.Processor类。

### 5.5 编写客户端

以下是客户端的实现：

```java
TTransport transport = new TSocket("localhost", 9090);
transport.open();

TProtocol protocol = new TBinaryProtocol(transport);
CrossPlatformService.Client client = new CrossPlatformService.Client(protocol);

boolean result = client.ping();

transport.close();
```

从客户的角度来看，这些操作非常相似。

首先，定义传输并将其指向我们的服务器实例，然后选择合适的协议。唯一的区别在于，这里我们初始化了客户端实例，该实例同样由Thrift生成，即CrossPlatformService.Client类。

由于它基于.thrift文件定义，因此我们可以直接调用其中描述的方法。在这个特定的例子中，client.ping()将对服务器进行远程调用，服务器将返回true。

## 6. 总结

在本文中，我们向你展示了使用Apache Thrift的基本概念和步骤，并展示了如何创建利用Thrift库的工作示例。