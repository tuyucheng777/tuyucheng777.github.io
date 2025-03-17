---
layout: post
title:  在Java中从字符串生成相同的UUID
category: java
copyright: java
excerpt: Java UUID
---

## 1. 概述

我们经常需要在应用程序中为各种目的生成唯一标识符，生成唯一标识符的一种常用方法是通用唯一标识符([UUID](https://www.baeldung.com/java-uuid))。

在本教程中，我们将探讨如何在Java中从字符串生成相同的UUID。

## 2. 问题介绍

当我们谈论从字符串生成UUID时，可能有两种情况：

- 场景1：输入字符串采用标准UUID字符串格式
- 场景2：给定的字符串是自由格式的字符串

接下来，我们将仔细研究如何从字符串生成UUID对象。当然，我们将介绍这两种情况。

为简单起见，我们将使用单元测试断言来验证每种方法是否能够产生预期的结果。

## 3. 给定的字符串是标准UUID表示

**标准的UUID字符串格式由五组用连字符分隔的十六进制数字组成**，例如：

```java
String inputStr = "bbcc4621-d88f-4a94-ae2f-b38072bf5087";
```

在这个场景中，我们想要从给定的字符串中获取一个UUID对象。此外，**生成的UUID对象的字符串表示必须等于输入字符串**。换句话说，这意味着generatedUUID.toString()与输入字符串相同。

因此，准确地说，我们要“解析”标准UUID格式的输入字符串，并根据解析的值构造一个新的UUID对象。

为了实现这一点，我们可以使用UUID.fromString()方法。接下来，让我们编写一个测试来看看它是如何工作的：

```java
String inputStr = "bbcc4621-d88f-4a94-ae2f-b38072bf5087";

UUID uuid = UUID.fromString(inputStr);
UUID uuid2 = UUID.fromString(inputStr);
UUID uuid3 = UUID.fromString(inputStr);

assertEquals(inputStr, uuid.toString());

assertEquals(uuid, uuid2);
assertEquals(uuid, uuid3);
```

如上面的测试所示，我们只需调用UUID.fromString(inputStr)即可生成UUID对象，标准UUID类负责输入解析和UUID生成。

此外，在我们的测试中，我们从同一个输入字符串生成了多个UUID对象，事实证明，**输入字符串生成的所有UUID对象都是相等的**。

使用UUID.fromString()方法很方便，但需要注意的是，输入的字符串必须是标准的UUID格式。否则，该方法将抛出IllegalArgumentException：

```java
String inputStr = "I am not a standard UUID representation.";
assertThrows(IllegalArgumentException.class, () -> UUID.fromString(inputStr));
```

## 4. 给定的输入是自由格式的字符串

我们已经看到UUID.fromString()可以方便地从标准UUID格式字符串构造UUID对象，让我们看看如何从自由格式字符串生成UUID对象。

**UUID类为我们提供了nameUUIDFromBytes(byte[] name)方法来构造[版本3](https://www.baeldung.com/java-generate-alphanumeric-uuid#1-versions)(也称为基于名称的) UUID对象**。 

由于该方法仅接收字节数组(byte[])，我们需要将[输入字符串转换为字节数组](https://www.baeldung.com/java-string-to-byte-array)以使用UUID.nameUUIDFromBytes()：

```java
String inputStr = "I am not a standard UUID representation.";

UUID uuid = UUID.nameUUIDFromBytes(inputStr.getBytes());
UUID uuid2 = UUID.nameUUIDFromBytes(inputStr.getBytes());
UUID uuid3 = UUID.nameUUIDFromBytes(inputStr.getBytes());

assertTrue(uuid != null);

assertEquals(uuid, uuid2);
assertEquals(uuid, uuid3);
```

从上面的测试中我们可以看出，我们通过相同的输入字符串三次调用UUID.nameUUIDFromBytes()生成了三个UUID对象，并且这三个UUID彼此相等。

在内部，此方法**根据输入字节数组的[MD5哈希值](https://www.baeldung.com/java-md5)返回一个UUID对象**。因此，对于给定的输入名称，生成的UUID保证是唯一的。

另外值得一提的是，**通过UUID.nameUUIDFromBytes()方法生成的UUID对象是版本3的UUID**，我们可以使用version()方法来验证：

```java
UUID uuid = UUID.nameUUIDFromBytes(inputStr.getBytes());
...
assertEquals(3, uuid.version());
```

版本5 UUID使用[SHA-1(160位)哈希函数，而不是MD5](https://www.baeldung.com/cs/md5-vs-sha-algorithms)。如果需要版本5 UUID，我们可以直接使用在引入[版本3和5](https://www.baeldung.com/java-generate-alphanumeric-uuid#2-versions-3-and-5) UUID时创建的generateType5UUID(String name)方法：

```java
String inputStr = "I am not a standard UUID representation.";

UUID uuid = UUIDGenerator.generateType5UUID(inputStr);
UUID uuid2 = UUIDGenerator.generateType5UUID(inputStr);
UUID uuid3 = UUIDGenerator.generateType5UUID(inputStr);

assertEquals(5, uuid.version());

assertTrue(uuid != null);

assertEquals(uuid, uuid2);
assertEquals(uuid, uuid3);
```

## 5. 总结

在本文中，我们探讨了如何从字符串生成相同的UUID对象，我们根据输入格式介绍了两种场景：

- 标准UUID格式字符串：使用UUID.fromString()
- 自由格式字符串：使用UUID.nameUUIDFromBytes()