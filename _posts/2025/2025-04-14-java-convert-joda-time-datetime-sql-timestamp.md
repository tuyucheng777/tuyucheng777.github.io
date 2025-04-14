---
layout: post
title:  在org.joda.time.DateTime和java.sql.Timestamp之间转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在Java中处理时间戳是一项常见的任务，它使我们能够更有效地操作和显示日期和时间信息，尤其是在处理数据库或全局应用程序时。处理时间戳和时区的两个基本类是[java.sql.Timestamp](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/security/Timestamp.html)和org.joda.time.DateTime。

在本教程中，我们将研究在org.joda.time.DateTime和java.sql.Timestamp之间进行转换的各种方法。

## 2. 将java.sql.Timestamp转换为org.joda.time.DateTime

首先，我们将研究将java.sql.Timestamp转换为org.joda.time.DateTime的多种方法。

### 2.1 使用构造函数

将[java.sql.Timestamp](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/security/Timestamp.html)转换为org.joda.time.DateTime的最简单方法之一是使用构造函数，**在此方法中，我们将使用getTime()方法获取自Unix[纪元](https://www.baeldung.com/java-localdate-epoch#epoch-vs-localdate)以来的毫秒数，并将该值作为DateTime构造函数的输入**。

让我们看看如何使用getTime()方法：

```java
DateTime convertToDateTimeUsingConstructor(Timestamp timestamp) {
    return new DateTime(timestamp.getTime());
}
```

我们来看下面的测试代码：

```java
public void givenTimestamp_whenUsingConstructor_thenConvertToDateTime() {
    long currentTimeMillis = System.currentTimeMillis();
    Timestamp timestamp = new Timestamp(currentTimeMillis);
    DateTime expectedDateTime = new DateTime(currentTimeMillis);
    DateTime convertedDateTime = DateTimeAndTimestampConverter.convertToDateTimeUsingConstructor(timestamp);
    assertEquals(expectedDateTime, convertedDateTime);
}
```

### 2.2 使用Instant类

**将Instant类视为UTC时区中的单个时刻，最简单的方法是将其视为UTC时区中的单个时刻**。如果我们将时间视为一条线，则[Instant](https://www.baeldung.com/java-instant-vs-localdatetime)表示该线上的单个点。

本质上，Instant类只是计算相对于标准Unix纪元时间1970年1月1日00:00:00的秒数和纳秒数，该时间点用0秒和0纳秒表示，其他值都只是相对于该时间点的偏移量。

通过存储相对于此特定时间点的秒数和纳秒数，该类可以存储正负偏移量。换句话说，**Instant类可以表示纪元时间之前和之后的时间**。

让我们看看如何使用Instant类将Timestamp转换为DateTime：

```java
DateTime convertToDateTimeUsingInstant(Timestamp timestamp) {
    Instant instant = timestamp.toInstant();
    return new DateTime(instant.toEpochMilli());
}
```

**在上述方法中，我们使用Timestamp类的toInstant()方法将提供的时间戳转换为Instant值，该值表示UTC时间轴上的某个时刻**。然后，我们对Instant对象使用toEpcohMilli()方法获取自Unix纪元以来的毫秒数。

让我们使用系统当前的毫秒数来测试这个方法：

```java
@Test
public void givenTimestamp_whenUsingInstant_thenConvertToDateTime() {
    long currentTimeMillis = System.currentTimeMillis();
    Timestamp timestamp = new Timestamp(currentTimeMillis);
    DateTime expectedDateTime = new DateTime(currentTimeMillis);
    DateTime convertedDateTime = DateTimeAndTimestampConverter.convertToDateTimeUsingInstant(timestamp);
    assertEquals(expectedDateTime, convertedDateTime);
}
```

### 2.3 使用LocalDateTime类

[java.time](https://www.baeldung.com/java-8-date-time-intro)包是在[Java 8](https://www.baeldung.com/java-8-new-features)中引入的，它提供了现代的日期和时间API。[LocalDatеTime](https://www.baeldung.com/java-8-date-time-intro)是此包中的一个类，它可以存储和操作不同时区的数据和时间，让我们来看看这种方法：

```java
DateTime convertToDateTimeUsingLocalDateTime(Timestamp timestamp) {
    LocalDateTime localDateTime = timestamp.toLocalDateTime();
    return new DateTime(localDateTime.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli());
}
```

Timestamp类的toLocalDateTime()方法将Timestamp转换为LocalDateTime，后者表示不带时区信息的日期和时间。让我们来看看这种方法：

```java
@Test
public void givenTimestamp_whenUsingLocalDateTime_thenConvertToDateTime() {
    long currentTimeMillis = System.currentTimeMillis();
    Timestamp timestamp = new Timestamp(currentTimeMillis);
    DateTime expectedDateTime = new DateTime(currentTimeMillis);
    DateTime convertedDateTime = DateTimeAndTimestampConverter.convertToDateTimeUsingLocalDateTime(timestamp);
    assertEquals(expectedDateTime, convertedDateTime);
}
```

## 3. 将org.joda.time.DateTime转换为java.sql.Timestamp

现在，我们将研究将org.joda.time.DateTime转换为java.sql.Timestamp的多种方法。

### 3.1 使用构造函数

我们也可以使用Timestamp构造函数将org.joda.time.DateTime转换为java.sql.Timestamp，在这里，我们将使用DateTime类的getMillis()方法，该方法将返回自Unix纪元以来的毫秒数，然后将此值提供给Timestamp构造函数。

让我们看看如何使用getMillis()方法：

```java
Timestamp convertToTimestampUsingConstructor(DateTime dateTime) {
    return new Timestamp(dateTime.getMillis());
}
```

现在，让我们测试一下这种方法：

```java
@Test
public void givenDateTime_whenUsingConstructor_thenConvertToTimestamp() {
    long currentTimeMillis = System.currentTimeMillis();
    DateTime dateTime = new DateTime(currentTimeMillis);
    Timestamp expectedTimestamp = new Timestamp(currentTimeMillis);
    Timestamp convertedTimestamp = DateTimeAndTimestampConverter.convertToTimestampUsingConstructor(dateTime);
    assertEquals(expectedTimestamp, convertedTimestamp);
}
```

### 3.2 使用Instant类

让我们看看如何使用Instant类将DateTime转换为java.sql.Timestamp：

```java
Timestamp convertToTimestampUsingInstant(DateTime dateTime) {
    Instant instant = Instant.ofEpochMilli(dateTime.getMillis());
    return Timestamp.from(instant);
}
```

在上述方法中，我们通过提供自Unix纪元以来的毫秒数，将提供的dateTime转换为Instant对象。之后，我们使用Timestamp的构造函数从Instant对象中获取Timestamp值。

我们来看下面的测试代码：

```java
@Test
public void givenDateTime_whenUsingInstant_thenConvertToTimestamp() {
    long currentTimeMillis = System.currentTimeMillis();
    DateTime dateTime = new DateTime(currentTimeMillis);
    Timestamp expectedTimestamp = new Timestamp(currentTimeMillis);
    Timestamp convertedTimestamp = DateTimeAndTimestampConverter.convertToTimestampUsingInstant(dateTime);
    assertEquals(expectedTimestamp, convertedTimestamp);
}
```

### 3.3 使用LocalDateTime类

让我们使用LocalDateTime类将DateTime转换为java.sql.Timestamp：


```java
Timestamp convertToTimestampUsingLocalDateTime(DateTime dateTime) {
    Instant instant = Instant.ofEpochMilli(dateTime.getMillis());
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
    return Timestamp.valueOf(localDateTime);
}
```

在上述方法中，我们通过提供自Unix纪元以来的毫秒数，将提供的dateTime转换为Instant，**LocalDateTime表示不带时区的日期和时间**。之后，我们使用ZoneId.systemDefault()检索系统默认[时区](https://www.baeldung.com/java-set-date-time-zone)，以确保使用本地时区设置创建LocalDateTime。

现在，让我们运行测试：

```java
@Test
public void givenDateTime_whenUsingLocalDateTime_thenConvertToTimestamp() {
    long currentTimeMillis = System.currentTimeMillis();
    DateTime dateTime = new DateTime(currentTimeMillis);
    Timestamp expectedTimestamp = new Timestamp(currentTimeMillis);
    Timestamp convertedTimestamp = DateTimeAndTimestampConverter.convertToTimestampUsingLocalDateTime(dateTime);
    assertEquals(expectedTimestamp, convertedTimestamp);
}
```

## 4. 总结

在本快速教程中，我们学习了如何在Java中将org.joda.time.DateTime转换为java.sql.Timestamp类。