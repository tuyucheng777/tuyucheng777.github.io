---
layout: post
title:  将String转换为Instant
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本快速教程中，我们将讲解如何使用java.time包中的类将Java中的String转换为Instant时间。首先，我们将使用LocalDateTime类实现解决方案。然后，我们将使用Instant类获取某个时区内的时间。

## 2. 使用LocalDateTime类

**java.time.LocalDateTime表示不带时区的日期和/或时间**，它是一个本地时间对象，这意味着它仅在特定上下文中有效，并且不能在此上下文之外使用(此上下文通常是指代码执行的机器)。

要从String中获取时间，我们可以使用DateTimeFormatter创建一个格式化对象，并将此格式化程序传递给LocalDateTime的parse方法。此外，我们还可以定义自己的格式化程序，或使用DateTimeFormatter类提供的预定义格式化程序。

让我们看看如何使用LocalDateTime.parse()从String中获取时间：

```java
String stringDate = "09:15:30 PM, Sun 10/09/2022"; 
String pattern = "hh:mm:ss a, EEE M/d/uuuu"; 
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern, Locale.US); 
LocalDateTime localDateTime = LocalDateTime.parse(stringDate, dateTimeFormatter);
```

在上面的例子中，我们使用LocalDateTime类(该类是表示带时间的日期的标准类)来解析日期字符串，我们也可以使用java.time.LocalDate来表示仅带时间的日期。

## 3. 使用Instant类

**java.time.Instant类是[Date-Time API](https://www.baeldung.com/java-8-date-time-intro)的主要类之一，它封装了时间轴上的一个点**。此外，它与java.util.Date类类似，但精度为纳秒。

在下一个示例中，我们将使用先前的LocalDateTime来获取具有指定ZoneId的时刻：

```java
String stringDate = "09:15:30 PM, Sun 10/09/2022"; 
String pattern = "hh:mm:ss a, EEE M/d/uuuu"; 
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern, Locale.US); 
LocalDateTime localDateTime = LocalDateTime.parse(stringDate, dateTimeFormatter); 
ZoneId zoneId = ZoneId.of("America/Chicago"); 
ZonedDateTime zonedDateTime = localDateTime.atZone(zoneId); 
Instant instant = zonedDateTime.toInstant();
```

在上面的例子中，我们首先创建一个ZoneId对象，用于标识一个时区，然后，我们提供了LocalDateTime和Instant之间的转换规则。

接下来，我们使用ZonedDateTime，它封装了包含时区及其相应偏移量的日期和时间。ZonedDateTime类是Date-Time API中与java.util.GregorianCalendar类最接近的类。最后，我们使用ZonedDateTime.toInstant()方法获取Instant值，该方法将时刻从时区调整为UTC。

## 4. 总结

在本快速教程中，我们解释了如何使用java.time包中的类将String转换为Java中的Instant。