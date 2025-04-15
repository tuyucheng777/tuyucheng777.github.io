---
layout: post
title:  在Java中从日期中提取年、月和日
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

**在这个简短的教程中，我们将学习如何从Java中给定的日期中提取年份、月份和日期**。

我们将讨论如何使用旧版java.util.Date类、java.sql.Timestamp以及Java 8的新日期时间库来提取这些值。

Java 8引入了一个全新的日期时间库，这有很多[好处](http://www.oracle.com/technetwork/articles/java/jf14-date-time-2125367.html)。除了其他优点之外，**新库还提供了更好的API来实现各种操作，例如从给定的Date中提取Year、Month、Day等**。

有关新日期时间库的更详细文章，请参阅[此处](https://www.baeldung.com/java-8-date-time-intro)。

## 2. 使用LocalDate

**新的java.time包包含几个可用于表示Date的类**。

每个类的不同之处在于它除了存储日期之外还存储其他信息。

基本的LocalDate仅包含日期信息，而LocalDateTime包含日期和时间信息。

类似地，更高级的类(例如OffsetDateTime和ZonedDateTime)分别包含有关UTC偏移量和时区的附加信息。

**无论如何，所有这些类都支持直接方法来提取年、月、日信息**。

让我们探索一下这些方法，从名为localDate的LocalDate实例中提取信息。

### 2.1 获取年份

为了提取年份，LocalDate提供一个getYear方法：

```java
localDate.getYear();
```

### 2.2 获取月份

类似地，要提取月份，我们将使用getMonthValue API：

```java
localDate.getMonthValue();
```

与Calendar不同，LocalDate中的月份从1开始索引；对于一月，这将返回1。

### 2.3 获取日期

最后，为了提取日期，我们有getDayOfMonth方法：

```java
localDate.getDayOfMonth();
```

## 3. 使用java.util.Date

对于给定的java.util.Date，要提取各个字段，例如Year、Month、Day等，我们需要做的第一步是将其转换为Calendar实例：

```java
Date date = // the date instance
Calendar calendar = Calendar.getInstance();
calendar.setTime(date);
```

一旦我们有了Calendar实例，我们就可以直接调用它的get方法，并提供我们想要提取的特定字段。

我们可以使用Calendar中的常量来提取特定字段。

### 3.1 获取年份

要提取年份，我们可以通过传递Calendar.YEAR作为参数来调用get：

```java
calendar.get(Calendar.YEAR);
```

### 3.2 获取月份

类似地，我们可以通过将Calendar.MONTH作为参数来调用get来提取月份：

```java
calendar.get(Calendar.MONTH);
```

**请注意，Calendar中的月份是从0开始的**。对于一月，此方法返回0。

### 3.3 获取日期

最后，为了提取日期，我们通过传递Calendar.DAY_OF_MONTH作为参数来调用get：

```java
calendar.get(Calendar.DAY_OF_MONTH);
```

## 4. 使用java.sql.Timestamp

对于给定的java.sql.Timestamp，要提取各个字段，如Year、Month、Day等，我们需要做的第一步是将其转换为Calendar实例：

```java
Timestamp timestamp = // the timestamp instance
Calendar calendar = Calendar.getInstance();
calendar.setTime(timestamp);
```

一旦我们有了Calendar实例，我们就可以直接调用它的get方法，并提供我们想要提取的特定字段。

我们可以使用Calendar中的常量来提取特定字段，正如我们之前看到的。

## 5. 总结

在这篇简短的文章中，我们探讨了如何从Java中的日期中提取年、月、日的整数值。

我们学习了如何使用旧的Date和Calendar类、Timestamp以及Java 8的新日期时间库来提取这些值。