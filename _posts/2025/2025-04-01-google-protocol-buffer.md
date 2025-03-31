---
layout: post
title:  Google Protocol Buffer简介
category: libraries
copyright: libraries
excerpt: Protocol Buffer
---

## 1. 概述

在本文中，我们将介绍[Google Protocol Buffer](https://developers.google.com/protocol-buffers/)(protobuf)，这是一种众所周知的与语言无关的二进制数据格式。我们可以使用协议定义文件，然后使用该协议，我们可以生成Java、C++、C#、Go或Python等语言的代码。

这是该格式本身的介绍性文章；如果你想了解如何在Spring Web应用程序中使用该格式，请查看[本文](https://www.baeldung.com/spring-rest-api-with-protocol-buffers)。

## 2. 定义Maven依赖

要在Java中使用协议缓冲区，我们需要向[protobuf-java](https://mvnrepository.com/artifact/com.google.protobuf/protobuf-java)添加Maven依赖：

```xml
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>${protobuf.version}</version>
</dependency>

<properties>
    <protobuf.version>3.2.0</protobuf.version>
</properties>
```

## 3. 定义协议

我们先来看一个例子，我们可以用protobuf格式定义一个非常简单的协议：

```protobuf
message Person {
    required string name = 1;
}
```

这是一个Person类型的简单消息协议，只有一个必填字段-字符串类型的name。

让我们看一下定义协议的更复杂的例子，假设我们需要以protobuf格式存储人员详细信息：

```protobuf
package protobuf;

option java_package = "cn.tuyucheng.taketoday.protobuf";
option java_outer_classname = "AddressBookProtos";

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  repeated string numbers = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

我们的协议包含两种类型的数据：Person和AddressBook。生成代码后(后面部分将详细介绍)，这些类将成为AddressBookProtos类中的内部类。

当我们想要定义一个必需的字段时-这意味着创建一个没有这个字段的对象将导致异常，我们需要使用一个required关键字。

使用optional关键字创建字段意味着这个字段不需要设置，repeated关键字是一个可变大小的数组类型。

所有字段都已编入索引-标有数字1的字段将作为二进制文件中的第一个字段保存。标有2的字段将作为下一个字段保存，依此类推。这使我们能够更好地控制字段在内存中的布局方式。

## 4. 从Protobuf文件生成Java代码

**一旦我们定义了一个文件，我们就可以从中生成代码**。

首先，我们需要在机器上[安装protobuf](https://github.com/google/protobuf/releases)。安装完成后，我们可以通过执行protoc命令来生成代码：

```shell
protoc -I=. --java_out=. addressbook.proto
```

protoc命令将从我们的addressbook.proto文件生成Java输出文件。-I选项指定proto文件所在的目录，java -out指定将创建生成的类的目录。

生成的类将包含我们定义的消息的Setter、Getter、构造函数和构建器，它还将包含一些实用方法，用于保存protobuf文件并将其从二进制格式反序列化为Java类。

## 5. 创建Protobuf定义消息的实例

我们可以轻松地使用生成的代码来创建Person类的Java实例：

```java
String email = "j@tuyucheng.com";
int id = new Random().nextInt();
String name = "Michael Program";
String number = "01234567890";
AddressBookProtos.Person person = AddressBookProtos.Person.newBuilder()
    .setId(id)
    .setName(name)
    .setEmail(email)
    .addNumbers(number)
    .build();

assertEquals(person.getEmail(), email);
assertEquals(person.getId(), id);
assertEquals(person.getName(), name);
assertEquals(person.getNumbers(0), number);
```

我们可以通过对所需消息类型使用newBuilder()方法创建一个流式的构建器，设置所有必填字段后，我们可以调用build()方法来创建Person类的实例。

## 6. 序列化和反序列化Protobuf

一旦创建了Person类的实例，我们就想将其以与所创建协议兼容的二进制格式保存在磁盘上。假设我们想创建AddressBook类的实例并将一个人添加到该对象。

接下来，我们要将该文件保存到磁盘上-我们可以使用自动生成的代码中一个writeTo()实用方法：

```java
AddressBookProtos.AddressBook addressBook = AddressBookProtos.AddressBook.newBuilder().addPeople(person).build();
FileOutputStream fos = new FileOutputStream(filePath);
addressBook.writeTo(fos);
```

执行该方法后，我们的对象将被序列化为二进制格式并保存在磁盘上。要从磁盘加载该数据并将其反序列化回AddressBook对象，我们可以使用mergeFrom()方法：

```java
AddressBookProtos.AddressBook deserialized = AddressBookProtos.AddressBook.newBuilder()
    .mergeFrom(new FileInputStream(filePath)).build();
 
assertEquals(deserialized.getPeople(0).getEmail(), email);
assertEquals(deserialized.getPeople(0).getId(), id);
assertEquals(deserialized.getPeople(0).getName(), name);
assertEquals(deserialized.getPeople(0).getNumbers(0), number);
```

## 7. 总结

在这篇简短的文章中，我们介绍了一种以二进制格式描述和存储数据的标准-Google Protocol Buffer。

我们创建了一个简单的协议，创建了符合定义协议的Java实例。接下来，我们了解了如何使用protobuf序列化和反序列化对象。