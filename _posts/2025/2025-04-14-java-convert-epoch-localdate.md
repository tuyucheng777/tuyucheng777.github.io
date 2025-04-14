---
layout: post
title:  将纪元时间转换为LocalDate和LocalDateTime
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

[纪元时间](https://www.baeldung.com/linux/epoch-time)(又称Unix时间)是一种将日期和时间表示为单个数值的系统，**它测量自1970年1月1日00:00:00(协调世界时，UTC)以来经过的毫秒数，纪元时间因其简单易用而被广泛应用于计算机系统和编程语言**。

在本教程中，我们将探讨以毫秒为单位的纪元时间到[LocalDate和LocalDateTime](https://www.baeldung.com/java-8-date-time-intro#localDate)的转换。

## 2. 将纪元时间转换为LocalDate

**要将纪元时间转换为LocalDate，我们需要将纪元时间(以毫秒为单位)转换为Instant对象**。

[Instant](https://www.baeldung.com/java-instant-vs-localdatetime)代表UTC时区时间线上的一个点：

```java
long epochTimeMillis = 1624962431000L; // Example epoch time in milliseconds
Instant instant = Instant.ofEpochMilli(epochTimeMillis);
```

一旦我们有了Instant对象，就可以通过使用atZone()方法指定时区并提取日期部分将其转换为LocalDate对象：

```java
ZoneId zoneId = ZoneId.systemDefault(); // Use the system default time zone
LocalDate localDate = instant.atZone(zoneId).toLocalDate();
```

最后，我们可以以人类可读的格式输出转换后的LocalDate对象：

```java
System.out.println(localDate); // Output: 2021-06-29
```

**我们可以使用[DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter)类的特定模式来格式化日期**：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String formattedDate = localDate.format(formatter);
System.out.println(formattedDate); // Output: 2021-06-29
```

我们可以根据需求选择不同的模式。

以下是该脚本的表示：

```java
long epochTimeMillis = 1624962431000L; // Example epoch time in milliseconds
Instant instant = Instant.ofEpochMilli(epochTimeMillis);

ZoneId zoneId = ZoneId.systemDefault(); // Use the system default time zone
LocalDate localDate = instant.atZone(zoneId).toLocalDate();

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String formattedDate = localDate.format(formatter);
System.out.println(formattedDate); // Output: 2021-06-29
```

使用这4个步骤，我们可以轻松地将毫秒级的纪元时间转换为LocalDate，甚至可以指定输出的格式。

## 3. 将纪元时间转换为LocalDateTime

将纪元时间(以毫秒为单位)转换为LocalDateTime的步骤与上面的LocalDate示例类似，唯一的区别是我们需要导入LocalDateTime类。

综上所述，这是转换为LocalDateTime的代码：

```java
long epochTimeMillis = 1624962431000L; // Example epoch time in milliseconds
Instant instant = Instant.ofEpochMilli(epochTimeMillis);

ZoneId zoneId = ZoneId.systemDefault(); // Use the system default time zone
LocalDateTime localDateTime = instant.atZone(zoneId).toLocalDateTime();

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String formattedDateTime = localDateTime.format(formatter);
System.out.println(formattedDateTime); // Output: 2021-06-29 12:13:51
```

**该代码将纪元时间(以毫秒为单位)转换为LocalDateTime，我们可以使用DateTimeFormatter类来格式化日期和时间**。

## 4. 总结

在本文中，我们探讨了将纪元时间(以毫秒为单位)转换为LocalDate和LocalDateTime，这是一个相当简单的过程，我们使用了DateTimeFormatter类将输出转换为特定的日期或时间格式。