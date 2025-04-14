---
layout: post
title:  在Java中获取当前日期和时间
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在本教程中，我们将探索如何在Java 8+及更早版本的环境中处理日期和时间。我们将首先介绍现代Java 8+的java.time包，然后介绍Java 8之前处理日期的传统方法。

## 2. 当前日期

首先，让我们使用java.time.LocalDate获取当前系统日期：

```java
LocalDate localDate = LocalDate.now();
```

要获取任何其他时区的日期，我们可以使用LocalDate.now(ZoneId)：

```java
LocalDate localDate = LocalDate.now(ZoneId.of("GMT+02:30"));
```

我们还可以使用java.time.LocalDateTime来获取LocalDate的实例：

```java
LocalDateTime localDateTime = LocalDateTime.now();
LocalDate localDate = localDateTime.toLocalDate();
```

## 3. 当前时间

使用java.time.LocalTime，让我们检索当前系统时间：

```java
LocalTime localTime = LocalTime.now();
```

要获取特定时区的当前时间，我们可以使用LocalTime.now(ZoneId)：

```java
LocalTime localTime = LocalTime.now(ZoneId.of("GMT+02:30"));
```

我们还可以使用java.time.LocalDateTime来获取LocalTime的实例：

```java
LocalDateTime localDateTime = LocalDateTime.now();
LocalTime localTime = localDateTime.toLocalTime();
```

## 4. 当前时间戳

使用java.time.Instant获取Java纪元的时间戳，根据[JavaDoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/Instant.html)的说明，“纪元秒数”是从标准Java纪元1970-01-01T00:00:00Z开始测量的，其中纪元之后的时刻为正值：

```java
Instant instant = Instant.now();
long timeStampMillis = instant.toEpochMilli();
```

我们可以得到纪元秒数：

```java
Instant instant = Instant.now();
long timeStampSeconds = instant.getEpochSecond();
```

## 5. Java 8之前使用日期和时间

对于仍在运行Java 8之前版本的系统，我们经常使用java.util和java.sql包进行日期和时间操作。

### 5.1 使用System时间

要获取自Unix纪元(1970年1月1日，00:00:00 GMT)以来的当前时间(以毫秒为单位)，我们可以使用currentTimeMillis()方法：

```java
long elapsedMilliseconds = System.currentTimeMillis();
```

为了获得更高的精度，我们可以使用System.nanoTime()以纳秒为单位返回当前时间，这对于计算间隔很有用：

```java
long elapsedNanosecondsStart = System.nanoTime();
long elapsedNanoseconds = System.nanoTime() - elapsedNanosecondsStart;
```

### 5.2 使用java.util.Date

Date类用于表示具有毫秒精度的特定日期和时间：

```java
Date currentUtilDate = new Date();
```

为了从字符串解析日期，我们可以使用SimpleDateFormat，它允许我们指定所需的日期格式：

```java
SimpleDateFormat dateFormatter = new SimpleDateFormat("dd-MM-yyyy HH:mm:ss");
Date customUtilDate = dateFormatter.parse("30-10-2024 10:11:12");
```

这种灵活性对于需要使用各种日期格式的应用程序非常有用。

### 5.3 使用java.util.Calendar

Calendar类在日期操作方面更加灵活，并且能够适应不同的语言环境。获取当前日期的方法如下：

```java
Calendar currentUtilCalendar = Calendar.getInstance();
```

我们可以使用以下方法轻松地将其转换为Date对象：

```java
Date currentDate = currentUtilCalendar.getTime();
```

此类对于执行算术运算特别有用，例如加或减天数。

### 5.4 使用java.sql.Date

java.sql.Date类表示不包含时区信息的日期，会截断时间部分。要获取当前SQL日期，请执行以下操作：

```java
Date currentSqlDate = new Date(System.currentTimeMillis());
```

我们可以使用valueOf()方法创建一个特定的日期，该方法需要格式yyyy-[mm\]-[d\]d：

```java
Date customSqlDate = Date.valueOf("2020-01-30");
```

此类通常用于仅与日期相关的数据库交互。

### 5.5 使用java.sql.Time

java.util.Time类捕获不带任何日期信息的时间(小时、分钟、秒)：

```java
Time currentSqlTime = new Time(System.currentTimeMillis());
```

要从字符串创建时间对象，我们可以使用Time.valueOf()：

```java
Time customSqlTime = Time.valueOf("10:11:12");
```

这在只需要时间组件的场景中很有用，例如调度事件。

### 5.6 使用java.sql.Timestamp

java.sql.Timestamp类结合了日期和时间，提供纳秒级精度。创建时间戳的方法如下：

```java
Timestamp currentSqlTimestamp = new Timestamp(System.currentTimeMillis());
```

对于自定义时间戳，我们使用valueOf()方法，格式为yyyy-[m\]m-[d\]d hh:mm:ss[.f...\]：

```java
Timestamp customSqlTimestamp = Timestamp.valueOf("2020-01-30 10:11:12.123456789");
```

## 6. 总结

在本文中，我们探讨了Java 8+ 之前和之后处理日期和时间的各种方法。我们学习了如何使用LocalDate、LocalTime和Instant检索当前日期、时间和时间戳。我们还深入研究了Java 8之前的类，包括java.util.Date、java.util.Calendar和java.sql.Date。