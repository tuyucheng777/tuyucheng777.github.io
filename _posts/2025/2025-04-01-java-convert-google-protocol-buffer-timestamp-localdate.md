---
layout: post
title:  将Google Protocol Buffer时间戳转换为LocalDate
category: libraries
copyright: libraries
excerpt: Google Protocol Buffer
---

## 1. 概述

[Protocol Buffer](https://www.baeldung.com/google-protocol-buffer)(protobuf)数据格式可帮助我们通过网络传输结构化数据，它独立于任何编程语言，并且适用于大多数编程语言，包括Java。

**protobuf Timestamp类型表示某个时间点，与任何特定时区无关**。时间是计算中的关键组成部分，有时我们需要将protobuf Timestamp转换为Java时间实例(例如LocalDate)，以便将其无缝集成到现有的Java代码库中。

在本教程中，我们将探讨将protobuf Timestamp实例转换为[LocalDate](https://www.baeldung.com/java-creating-localdate-with-values)类型的过程，使我们能够在Java应用程序中更有效地处理protobuf数据。

## 2. Maven依赖

要创建Timestamp的实例或使用从.proto文件生成的代码，我们需要在pom.xml中添加[protobuf-java](https://mvnrepository.com/artifact/com.google.protobuf/protobuf-java)依赖：

```xml
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>4.26.1</version>
</dependency>
```

该依赖提供了Timestamp类和其他与protobuf相关的类。

## 3. Timestamp类

protobuf Timestamp类表示自Unix纪元以来的时间点，时区或当地日历不会影响它。

**它表示某一时间点的秒数和纳秒数**，让我们看一个使用Java Instant对象计算当前时间戳的示例：

```java
Instant currentTimestamp = Instant.now();

Timestamp timestamp = Timestamp.newBuilder()
    .setSeconds(currentTimestamp.getEpochSecond())
    .setNanos(currentTimestamp.getNano())
    .build();
```

在上面的代码中，我们从Instant对象计算时间戳。首先，我们创建一个Instant对象，表示给定时间点的日期和时间。接下来，我们提取秒和纳秒并将它们传递给Timestamp实例。

## 4. 将Timestamp实例转换为LocalDate

**从Timestamp转换为LocalDate时，考虑时区及其与[UTC](https://www.baeldung.com/java-time-zones)的偏移量以准确表示本地日期至关重要**。要将Timestamp转换为LocalDate，让我们首先创建一个具有一组特定秒和纳秒的Timestamp实例：

```java
Timestamp ts = Timestamp.newBuilder()
    .setSeconds(1000000)
    .setNanos(77886600)
    .build();
```

由于[Instant](https://www.baeldung.com/java-instant-to-string)类是最适合表示某个时间点的类，因此让我们创建一个接收Timestamp作为参数并将其转换为LocalDate的方法：

```java
LocalDate convertToLocalDate(Timestamp timestamp) {
    Instant instant = Instant.ofEpochSecond(timestamp.getSeconds(), timestamp.getNanos());
    LocalDate time = instant.atZone(ZoneId.of("America/Montreal")).toLocalDate();
    return time;
}
```

在上面的代码中，我们使用ofEpochSecond()方法从Timestamp创建一个Instant对象，该方法将Timestamp中的秒和纳秒作为参数。

**然后，我们使用atZone()方法将Instant对象转换为LocalDate，这允许我们指定[时区](https://www.baeldung.com/java-set-date-time-zone)**。这很重要，因为它可以确保生成的LocalDate反映时区偏移量。

值得注意的是，我们还可以使用我们的默认系统时区：

```java
LocalDate time = instant.atZone(ZoneId.systemDefault()).toLocalDate();
```

让我们编写一个单元测试来断言逻辑：

```java
@Test
void givenTimestamp_whenConvertedToLocalDate_thenSuccess() {
    Timestamp timestamp = Timestamp.newBuilder()
        .setSeconds(1000000000)
        .setNanos(778866000)
        .build();
    LocalDate time = TimestampToLocalDate.convertToLocalDate(timestamp);
    assertEquals(LocalDate.of(2001, 9, 9), time);
}
```

在上面的代码中，我们断言转换后的LocalDate与预期结果相符。

## 5. 总结

在本教程中，我们学习了如何通过转换为Instant来表示某个时间点，从而将Timestamp转换为LocalDate。此外，我们还通过设置时区来处理可能的偏移，将Instance对象转换为LocalDate类型。