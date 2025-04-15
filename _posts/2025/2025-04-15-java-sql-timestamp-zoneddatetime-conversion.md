---
layout: post
title:  Java中在java.sql.Timestamp和ZonedDateTime之间进行转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在Java中处理时间戳是一项常见的任务，它使我们能够更有效地操作和显示日期和时间信息，尤其是在处理数据库或全局应用程序时。处理时间戳和时区的两个基本类是java.sql.Timestamp和[ZonedDateTime](https://www.baeldung.com/java-zoneddatetime-offsetdatetime#zoneddatetime)。

在本教程中，我们将研究在java.sql.Timestamp和ZonedDateTime之间转换的各种方法。

## 2. 将java.sql.Timestamp转换为ZonedDateTime

首先，我们将研究将java.sql.Timestamp转换为ZonedDateTime的多种方法。

### 2.1 使用Instant类

**最简单的方法是将Instant类视为UTC时区中的单个时刻**，如果我们将时间视为一条线，则Instant表示该线上的单个点。

本质上，Instant类只是计算相对于标准Unix纪元时间1970年1月1日00:00:00的秒数和纳秒数，该时间点用0秒和0纳秒表示，其他值都只是相对于该时间点的偏移量。

通过存储相对于此特定时间点的秒数和纳秒数，该类可以存储正负偏移量。换句话说，**Instant类可以表示纪元时间之前和之后的时间**。

让我们看看如何使用Instant类将时间戳转换为ZonedDateTime：

```java
ZonedDateTime convertToZonedDateTimeUsingInstant(Timestamp timestamp) {
    Instant instant = timestamp.toInstant();
    return instant.atZone(ZoneId.systemDefault());
}
```

在上述方法中，我们使用Timestamp类的toInstant()方法将提供的时间戳转换为Instant，该时间戳表示UTC时间轴上的某个时刻。然后，我们使用Instant对象上的atZone()方法将其与特定时区关联，我们使用通过ZoneId.systemDefault()获取的系统默认时区。

让我们使用系统默认时区(全球时区)来测试此方法：

```java
@Test
void givenTimestamp_whenUsingInstant_thenConvertToZonedDateTime() {
    Timestamp timestamp = Timestamp.valueOf("2024-04-17 12:30:00");
    ZonedDateTime actualResult = TimestampAndZonedDateTimeConversion.convertToZonedDateTimeUsingInstant(timestamp);
    ZonedDateTime expectedResult = ZonedDateTime.of(2024, 4, 17, 12, 30, 0, 0, ZoneId.systemDefault());
    Assertions.assertEquals(expectedResult.toLocalDate(), actualResult.toLocalDate());
    Assertions.assertEquals(expectedResult.toLocalTime(), actualResult.toLocalTime());
}
```

### 2.2 使用Calendar类

另一个解决方案是使用旧版Date API中的[Calendar](https://www.baeldung.com/java-gregorian-calendar)类，**该类提供了setTimeInMillis(long value)方法，我们可以使用它来将时间设置为给定的long值**：

```java
ZonedDateTime convertToZonedDateTimeUsingCalendar(Timestamp timestamp) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTimeInMillis(timestamp.getTime());
    return calendar.toInstant().atZone(ZoneId.systemDefault());
}
```

在上面的方法中，我们使用Calendar.getInstance()方法初始化了Calendar实例，**Calendar实例的时间设置为与Timestamp对象相同的时间。之后，我们在Calendar对象上使用toInstant()方法获取Instant值**。然后，我们再次在Instant对象上使用atZone()方法将其与特定时区关联，我们使用了通过ZoneId.systemDefault()获取的系统默认时区。 

我们来看下面的测试代码：

```java
@Test
void givenTimestamp_whenUsingCalendar_thenConvertToZonedDateTime() {
    Timestamp timestamp = Timestamp.valueOf("2024-04-17 12:30:00");
    ZonedDateTime actualResult = TimestampAndZonedDateTimeConversion.convertToZonedDateTimeUsingCalendar(timestamp);
    ZonedDateTime expectedResult = ZonedDateTime.of(2024, 4, 17, 12, 30, 0, 0, ZoneId.systemDefault());
    Assertions.assertEquals(expectedResult.toLocalDate(), actualResult.toLocalDate());
    Assertions.assertEquals(expectedResult.toLocalTime(), actualResult.toLocalTime());
}
```

### 2.3 使用LocalDateTime类

[java.time](https://www.baeldung.com/java-8-date-time-intro)包是在Java 8中引入的，它提供了现代的日期和时间API。LocalDatеTime是此包中的一个类，它可以存储和操作不同时区的数据和时间，让我们来看看这种方法：

```java
ZonedDateTime convertToZonedDateTimeUsingLocalDateTime(Timestamp timestamp) {
    LocalDateTime localDateTime = timestamp.toLocalDateTime();
    return localDateTime.atZone(ZoneId.systemDefault());
}
```

Timestamp类的toLocalDateTime()方法将Timestamp转换为LocalDateTime，后者表示不带时区信息的日期和时间。

让我们测试一下这种方法：

```java
@Test
void givenTimestamp_whenUsingLocalDateTime_thenConvertToZonedDateTime() {
    Timestamp timestamp = Timestamp.valueOf("2024-04-17 12:30:00");
    ZonedDateTime actualResult = TimestampAndZonedDateTimeConversion.convertToZonedDateTimeUsingLocalDateTime(timestamp);
    ZonedDateTime expectedResult = ZonedDateTime.of(2024, 4, 17, 12, 30, 0, 0, ZoneId.systemDefault());
    Assertions.assertEquals(expectedResult.toLocalDate(), actualResult.toLocalDate());
    Assertions.assertEquals(expectedResult.toLocalTime(), actualResult.toLocalTime());
}
```

### 2.4 使用Joda-Time类

[Joda-Time](https://www.baeldung.com/joda-time)是一个非常流行的Java日期和时间操作库，它提供了比标准[DateTime](https://www.baeldung.com/java-8-date-time-intro)类更加直观和灵活的API。

为了包含Joda-Time库的功能，我们需要从[Maven Central](https://mvnrepository.com/artifact/joda-time/joda-time)添加以下依赖：

```xml
<dependency> 
    <groupId>joda-time</groupId> 
    <artifactId>joda-time</artifactId> 
    <version>2.12.7</version> 
</dependency>
```

让我们看看如何使用Joda-Time类来实现这种转换：

```java
ZonedDateTime convertToZonedDateTimeUsingJodaTime(Timestamp timestamp) {
    DateTime dateTime = new DateTime(timestamp.getTime());
    return dateTime.toGregorianCalendar().toZonedDateTime();
}
```

在这个方法中，我们首先获取自纪元(1970-01-01T00:00:00Z)以来的毫秒数。然后，我们使用默认时区，根据获取的毫秒值创建一个新的DateTime对象。

接下来，我们将DateTime对象转换为GregorianCalendar，然后使用Joda-Time库的方法将GregorianCalendar转换为ZonedDateTime。

现在，让我们运行测试：

```java
@Test
void givenTimestamp_whenUsingJodaTime_thenConvertToZonedDateTime() {
    Timestamp timestamp = Timestamp.valueOf("2024-04-17 12:30:00");
    ZonedDateTime actualResult = TimestampAndZonedDateTimeConversion.convertToZonedDateTimeUsingJodaTime(timestamp);
    ZonedDateTime expectedResult = ZonedDateTime.of(2024, 4, 17, 12, 30, 0, 0, ZoneId.systemDefault());
    Assertions.assertEquals(expectedResult.toLocalDate(), actualResult.toLocalDate());
    Assertions.assertEquals(expectedResult.toLocalTime(), actualResult.toLocalTime());
}
```

## 3. 将ZonedDateTime转换为java.sql.Timestamp

现在，我们将研究将ZonedDateTime转换为java.sql.Timestamp的多种方法。

### 3.1 使用Instant类

让我们看看如何使用Instant类将ZonedDateTime转换为java.sql.Timestamp：

```java
Timestamp convertToTimeStampUsingInstant(ZonedDateTime zonedDateTime) {
    Instant instant = zonedDateTime.toInstant();
    return Timestamp.from(instant);
}
```

在上面的方法中，我们首先使用toInstant()方法将提供的ZonedDateTime对象转换为Instant。然后，我们使用Timestamp类的from()方法，通过将获得的Instant作为参数传递来创建Timestamp对象。

现在，让我们测试一下这种方法：

```java
@Test
void givenZonedDateTime_whenUsingInstant_thenConvertToTimestamp() {
    ZonedDateTime zonedDateTime = ZonedDateTime.of(2024, 4, 17, 12, 30, 0, 0, ZoneId.systemDefault());
    Timestamp actualResult = TimestampAndZonedDateTimeConversion.convertToTimeStampUsingInstant(zonedDateTime);
    Timestamp expectedResult = Timestamp.valueOf("2024-04-17 12:30:00");
    Assertions.assertEquals(expectedResult, actualResult);
}
```

### 3.2 使用LocalDateTime类

让我们使用LocalDateTime类将ZonedDateTime转换为java.sql.Timestamp：


```java
Timestamp convertToTimeStampUsingLocalDateTime(ZonedDateTime zonedDateTime) {
    LocalDateTime localDateTime = zonedDateTime.toLocalDateTime();
    return Timestamp.valueOf(localDateTime);
}
```

在上面的方法中，我们使用toLocalDateTime()方法将提供的ZonedDateTime对象转换为LocalDateTime对象，**LocalDateTime表示不带时区的日期和时间**。然后，我们将LocalDateTime对象作为参数传递，并使用Timestamp类的valueOf()方法创建并返回一个Timestamp对象。

现在，让我们运行测试：

```java
@Test
void givenZonedDateTime_whenUsingLocalDateTime_thenConvertToTimestamp() {
    ZonedDateTime zonedDateTime = ZonedDateTime.of(2024, 4, 17, 12, 30, 0, 0, ZoneId.systemDefault());
    Timestamp actualResult = TimestampAndZonedDateTimeConversion.convertToTimeStampUsingLocalDateTime(zonedDateTime);
    Timestamp expectedResult = Timestamp.valueOf("2024-04-17 12:30:00");
    Assertions.assertEquals(expectedResult, actualResult);
}
```

### 3.3 使用Joda-Time类

让我们看看如何使用Joda-Time类来实现这种转换：

```java
Timestamp convertToTimestampUsingJodaTime(ZonedDateTime zonedDateTime) {
    DateTime dateTime = new DateTime(zonedDateTime.toInstant().toEpochMilli());
    return new Timestamp(dateTime.getMillis());
}
```

在这个方法中，我们首先通过将ZonedDateTime对象转换为Instant来获取自纪元(1970-01-01T00:00:00Z)以来的毫秒数。然后，我们使用默认时区，根据获取的毫秒值创建一个新的DateTime对象。

类似地，我们创建一个Timestamp对象，它表示与DateTime对象相同的时间点。现在让我们测试一下这种方法：

```java
@Test
void givenZonedDateTime_whenUsingJodaDateTime_thenConvertToTimestamp() {
    ZonedDateTime zonedDateTime = ZonedDateTime.of(2024, 4, 17, 12, 30, 0, 0, ZoneId.systemDefault());
    Timestamp actualResult = TimestampAndZonedDateTimeConversion.convertToTimestampUsingJodaTime(zonedDateTime);
    Timestamp expectedResult = Timestamp.valueOf("2024-04-17 12:30:00");
    Assertions.assertEquals(expectedResult, actualResult);
}
```

## 4. 总结

在本快速教程中，我们学习了如何在Java中转换ZonedDateTime和java.sql.Timestamp类。