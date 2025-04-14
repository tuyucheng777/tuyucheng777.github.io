---
layout: post
title:  在Java中将Long时间戳转换为LocalDateTime
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在Java中处理时间戳是一项常见任务，它使我们能够更有效地操作和显示日期和时间信息，尤其是在处理数据库或外部API时。

**在本教程中，我们将研究如何将Long[时间戳](https://www.baeldung.com/java-string-to-timestamp)转换为[LocalDatеTime](https://www.baeldung.com/java-8-date-time-intro)对象**。

## 2. 理解Long时间戳和LocalDatеTime

### 2.1 Long时间戳

Long时间戳以自纪元以来的毫秒数来表示特定的时间点，具体来说，它是一个单一值，表示自1970年1月1日以来经过的时间。

此外，以这种格式处理时间戳对于计算来说很有效，但需要转换为可读的日期时间格式以供用户交互或显示目的。

**例如，Long值1700010123000L表示参考点2023-11-15 01:02:03**。

### 2.2 LocalDateTime

[java.time](https://www.baeldung.com/java-8-date-time-intro)包是在Java 8中引入的，它提供了现代的日期和时间API。LocalDatеTimе是此包中的一个类，可以存储和操作不同时区的数据和时间。

## 3. 使用Instant类

[Instant](https://www.baeldung.com/java-instant-vs-localdatetime)类表示一个时间点，并且可以轻松转换为其他日期和时间表示形式。因此，要将Long时间戳转换为LocalDatеTime对象，我们可以按如下方式使用Instant类：

```java
long timestampInMillis = 1700010123000L;
String expectedTimestampString = "2023-11-15 01:02:03";

@Test
void givenTimestamp_whenConvertingToLocalDateTime_thenConvertSuccessfully() {
    Instant instant = Instant.ofEpochMilli(timestampInMillis);
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.of("UTC"));

    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String formattedDateTime = localDateTime.format(formatter);

    assertEquals(expectedTimestampString, formattedDateTime);
}
```

在上面的测试方法中，我们用1700010123000L初始化一个Long类型的变量timestampInMillis。此外，我们使用Instant.ofEpochMilli(timestampInMillis)方法将Long类型的时间戳转换为Instant类型的时间戳。

随后，LocalDatеTime.ofInstant(instant,ZonеId.of("UTC"))方法使用UTC时区将Instant转换为LocalDatеTime。

## 4. 使用Joda-Time

[Joda-Time](https://www.baeldung.com/joda-time)是Java中流行的日期和时间操作库，它为标准Java日期和时间API提供了更直观的替代方案。

让我们探索如何使用Joda-Time将长时间戳转换为格式化的LocalDatеTime字符串：

```java
@Test
void givenJodaTime_whenGettingTimestamp_thenConvertToLong() {
    DateTime dateTime = new DateTime(timestampInMillis, DateTimeZone.UTC);
    org.joda.time.LocalDateTime localDateTime = dateTime.toLocalDateTime();

    org.joda.time.format.DateTimeFormatter formatter = DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss");
    String actualTimestamp = formatter.print(localDateTime);

    assertEquals(expectedTimestampString, actualTimestamp);
}
```

在这里，我们根据提供的Long值实例化DateTime对象。此外，DateTimeZone.UTC方法将时区显式定义为DateTime对象的协调通用时间(UTC)。随后，toLocalDateTimе()函数将DateTime对象无缝转换为LocalDateTime对象，从而保持与时区无关的表示。

最后，我们利用名为formatter的DatеTimeFormattеr将LocalDatеTime对象按照指定的模式转换为字符串。

## 5. 总结

总而言之，在Java中将Long时间戳转换为LocalDatеTime对象的过程涉及使用Instant类来准确表示时间，这允许有效地操作和显示日期和时间信息，确保与数据库或外部API的无缝交互。