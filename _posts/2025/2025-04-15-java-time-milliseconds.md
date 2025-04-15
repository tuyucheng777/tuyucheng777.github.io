---
layout: post
title:  在Java中将时间转换为毫秒
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本快速教程中，**我们将说明在Java中将时间转换为Unix纪元毫秒的多种方法**。

更具体地说，我们将使用：

- 核心Java的java.util.Date和Calendar
- Java 8的日期和时间API
- Joda-Time库

## 2. 核心Java

### 2.1 使用Date

首先，让我们定义一个millis属性来保存毫秒的随机值：

```java
long millis = 1556175797428L; // April 25, 2019 7:03:17.428 UTC
```

我们将使用这个值来初始化我们的各种对象并验证结果。

接下来，让我们从一个Date对象开始：

```java
Date date = // implementation details
```

现在，我们**只需调用getTime()方法即可将日期转换为毫秒**：

```java
Assert.assertEquals(millis, date.getTime());
```

### 2.2 使用Calendar

同样，如果我们有一个Calendar对象，我们可以使用getTimeInMillis()方法：

```java
Calendar calendar = // implementation details
Assert.assertEquals(millis, calendar.getTimeInMillis());
```

## 3. Java 8日期时间API

### 3.1 使用Instant

简单来说，[Instant](https://www.baeldung.com/current-date-time-and-timestamp-in-java-8)是Java纪元时间轴上的一个点。

我们可以从Instant获取当前时间(以毫秒为单位)：

```java
java.time.Instant instant = // implementation details
Assert.assertEquals(millis, instant.toEpochMilli());
```

因此，toEpochMilli()方法返回与我们之前定义的相同的毫秒数。

### 3.2 使用LocalDateTime

类似地，我们可以使用Java 8的[日期和时间API](https://www.baeldung.com/java-8-date-time-intro)将LocalDateTime转换为毫秒：

```java
LocalDateTime localDateTime = // implementation details
ZonedDateTime zdt = ZonedDateTime.of(localDateTime, ZoneId.systemDefault());
Assert.assertEquals(millis, zdt.toInstant().toEpochMilli());
```

首先，我们创建了当前日期的实例，然后，我们使用toEpochMilli()方法将ZonedDateTime转换为毫秒。

我们知道，LocalDateTime不包含时区信息，换句话说，**我们无法直接从LocalDateTime实例获取毫秒数**。

## 4. Joda-Time

虽然Java 8增加了Joda-Time的许多功能，但如果我们使用的是Java 7或更早版本，我们可能希望使用此选项。

### 4.1 使用Instant

首先，我们可以使用getMillis()方法从[Joda-Time](https://www.baeldung.com/joda-time)Instant类实例中获取当前系统毫秒数：

```java
Instant jodaInstant = // implementation details
Assert.assertEquals(millis, jodaInstant.getMillis());
```

### 4.2 使用DateTime

此外，如果我们有一个Joda-Time DateTime实例：

```java
DateTime jodaDateTime = // implementation details
```

然后我们可以使用getMillis()方法检索毫秒数：

```java
Assert.assertEquals(millis, jodaDateTime.getMillis());
```

## 5. 总结

本文演示了如何在Java中将时间转换为毫秒。