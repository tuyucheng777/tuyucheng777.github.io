---
layout: post
title:  将Joda-Time DateTime转换为Date
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

[Joda-Time](https://www.baeldung.com/joda-time)是一个非常流行的Java日期和时间操作库，它提供了比标准[DateTime](https://www.baeldung.com/java-8-date-time-intro)类更直观、更灵活的API。

**在本教程中，我们将研究如何将Joda-Time DateTime对象转换为标准Java Date对象，反之亦然**。

## 2. 设置Joda-Time

首先，我们应该确保项目包含[joda-time](https://mvnrepository.com/artifact/joda-time/joda-time)库：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.6</version>
</dependency>
```

或者，我们可以[下载](https://jar-download.com/artifacts/joda-time/joda-time)jar文件并将其放在我们的classpath中。

## 3. 将Joda-Time DateTime转换为Java Date

要将Joda-Time DateTime对象转换为标准Java Date，我们使用名为toDate()的方法，以下是一个简单的示例：

```java
@Test
public void givenJodaDateTime_whenConvertingToJavaDate_thenConversionIsCorrect() {
    DateTime jodaDateTime = new DateTime();
    java.util.Date javaDate = jodaDateTime.toDate();
    assertEquals(jodaDateTime.getMillis(), javaDate.getTime());
}
```

在这个测试方法中，我们从Joda-Time创建了一个名为jodaDateTime的DateTime新实例。随后，我们在这个Joda DateTime实例上调用toDate()方法来获取相应的java.util.Date对象。

该测试使用assertEquals方法执行，该方法断言从原始Joda DateTime对象检索到的时间(以毫秒为单位)等于从通过java.util.Date创建的新DateTime对象获取的时间。

## 4. 将Java Date转换为Joda-Time DateTime

将普通的Java Date对象转换为Joda-Time DateTime也很简单，我们可以使用为java.util.Date参数设计的DateTime构造函数，如下所示：

```java
@Test
public void givenJavaDate_whenConvertingToJodaDateTime_thenConversionIsCorrect() {
    java.util.Date javaDate = new java.util.Date();
    DateTime jodaDateTime = new DateTime(javaDate);
    assertEquals(javaDate.getTime(), jodaDateTime.getMillis());
}
```

在上述测试方法中，我们主动实例化了一个新的java.util.Date对象，用于表示当前日期和时间。随后，我们使用提供的Java Date创建一个对应的Joda DateTime对象。实际的验证使用assertEquals方法进行，该方法验证从原始java.util.Date对象获取的毫秒数是否等于Joda DateTime对象所表示的时间。

## 5. 总结

总之，在Java中处理日期和时间时，通常的操作之一是在Joda-Time DateTime对象和标准Java Date之间进行转换。