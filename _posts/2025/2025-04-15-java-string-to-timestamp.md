---
layout: post
title:  在String和Timestamp之间转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

Timestamp是Java中少数几个遗留的日期时间对象之一。

在本教程中，我们将了解如何将字符串值解析为Timestamp对象以及如何将Timestamp对象格式化为字符串。

由于Timestamp依赖于Java专有格式，我们将了解如何有效地适应。

## 2. 将String解析为Timestamp

### 2.1 标准格式

**将String解析为Timestamp的最简单方法是其valueOf方法**：

```java
Timestamp.valueOf("2018-11-12 01:02:03.123456789")
```

当我们的String采用JDBC时间戳格式yyyy-m[m\]-d[d\]hh:mm: ss[.f...\]时-那么它就非常简单。

我们可以这样解释这个模式：

|  模式   |                 描述                  |        例子         |
|:-----:|:-----------------------------------:|:-----------------:|
| yyyy  |            代表年份，必须为四位数字             |       2018        |
| m[\]  |     对于月份部分，我们必须有一个或两个数字(从1到12)      |       1、11        |
| d[d\] |    对于月份中的日期值，我们必须有一个或两个数字(从1到31)    |       7、12        |
|  hh   |         代表一天中的小时，允许的值从0到23          |       01、16       |
|  mm   |         代表一小时中的分钟数，允许值从0到59         |       02、45       |
|  sss  |         代表一分钟内的秒数，允许值从0到59          |       03、52       |
|  [.f...\]  |表示可选的秒数分数，精度可达纳秒，因此允许的值是从0到999999999 | 12、1567、123456789 |

### 2.2 替代格式

现在，如果它不是JDBC时间戳格式，那么幸运的是，valueOf也采用LocalDateTime实例。

**这意味着我们可以采用任何格式的日期**，我们只需要先将其[转换为LocalDateTime](https://www.baeldung.com/java-string-to-date)：

```java
String pattern = "MMM dd, yyyy HH:mm:ss.SSSSSSSS";
String timestampAsString = "Nov 12, 2018 13:02:56.12345678";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
LocalDateTime localDateTime = LocalDateTime.from(formatter.parse(timestampAsString));
```

然后我们可以使用之前的valueOf：

```java
Timestamp timestamp = Timestamp.valueOf(localDateTime);
assertEquals("2018-11-12 13:02:56.12345678", timestamp.toString());
```

顺便注意，**与Date对象不同，Timestamp对象能够存储秒的几分之一**。

## 3. 将Timestamp格式化为String

要格式化Timestamp，我们将面临同样的挑战，因为它的默认格式是专有的JDBC时间戳格式：

```java
assertEquals("2018-11-12 13:02:56.12345678", timestamp.toString());
```

但是，再次使用中间转换，我们可以将结果String格式化为不同的日期和时间模式，例如ISO-8601标准：

```java
Timestamp timestamp = Timestamp.valueOf("2018-12-12 01:02:03.123456789");
DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
 
String timestampAsString = formatter.format(timestamp.toLocalDateTime());
assertEquals("2018-12-12T01:02:03.123456789", timestampAsString);
```

## 4. 总结

在本文中，我们了解了如何在Java中在String和Timestamp对象之间进行转换。此外，我们还了解了如何使用LocalDateTime转换作为中间步骤，以便在不同的日期和时间模式之间进行转换。