---
layout: post
title:  Java中在ZonedDateTime和Date之间进行转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在使用处理日期和时间的Java应用程序时，了解如何在不同类型之间进行转换通常至关重要。

在本教程中，我们将探讨如何在Java中在ZonedDateTime和Date之间进行转换。

## 2. 理解ZonedDateTime和Date

在深入研究转换之前，让我们先了解Java中ZonedDateTime和Date的核心概念，这将帮助我们了解何时以及为何在这两种类型之间进行转换。

### 2.1 ZonedDateTime

ZonedDateTime是Java 8中引入的[Java日期和时间API](https://www.baeldung.com/java-8-date-time-intro)的一部分，ZonedDateTime对象包含：

- 当地日期和时间(年、月、日、时、分、秒、纳秒)
- 时区(由ZoneId表示)
- 与UTC/格林威治的偏移量

这使得ZonedDateTime对于需要处理不同时区的应用程序非常有用。

### 2.2 Date

Date类是Java早期版本中原始日期和时间API的一部分，**它表示从UNIX纪元(1970年1月1日00:00:00 GMT)开始的特定时间点，精度为毫秒**。

Date不包含任何时区信息，**它本质上与时区无关，始终表示UTC时间点**。如果开发人员错误地将Date解释为表示本地时区的时间，则可能会造成混淆。

## 3. 将ZonedDateTime转换为Date

要将ZonedDateTime转换为Date，我们首先需要将ZonedDateTime转换为Instant，因为ZonedDateTime和Instant都代表时间轴上适合转换的点。让我们看看怎么做：

```java
public static Date convertToDate(ZonedDateTime zonedDateTime) {
    return Date.from(zonedDateTime.toInstant());
}
```

## 4. 将Date转换为ZonedDateTime

从日期转换回ZonedDateTime也涉及从日期获取瞬间，然后指定我们希望ZonedDateTime具有的时区：

```java
public static ZonedDateTime convertToZonedDateTime(Date date, ZoneId zone) {
    return date.toInstant().atZone(zone);
}
```

在这个转换过程中，**指定时区(ZoneId)非常重要，因为Date对象不包含时区信息**，我们可以使用静态方法ZoneId.systemDefault()获取系统默认的ZoneId。

## 5. 测试

让我们使用JUnit编写一些单元测试来确保我们的转换方法正确：

```java
@Test
public void givenZonedDateTime_whenConvertToDate_thenCorrect() {
    ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("UTC"));
    Date date = DateAndZonedDateTimeConverter.convertToDate(zdt);
    assertEquals(Date.from(zdt.toInstant()), date);
}

@Test
public void givenDate_whenConvertToZonedDateTime_thenCorrect() {
    Date date = new Date();
    ZoneId zoneId = ZoneId.of("UTC");
    ZonedDateTime zdt = DateAndZonedDateTimeConverter.convertToZonedDateTime(date, zoneId);
    assertEquals(date.toInstant().atZone(zoneId), zdt);
}
```

## 6. 总结

一旦我们理解了如何处理转换点(即Instant和ZoneId)，在Java中ZonedDateTime和Date之间的转换就很简单了。虽然ZonedDateTime提供了更强大的功能和灵活性，但Date在需要与旧Java代码库集成的场景中仍然很有用。