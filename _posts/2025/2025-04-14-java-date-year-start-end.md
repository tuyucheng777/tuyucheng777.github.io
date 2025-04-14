---
layout: post
title:  如何使用Java获取一年的开始和结束日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

Java 8引入了新的[日期时间API](https://www.baeldung.com/migrating-to-java-8-date-time-api)，使Java中的日期和时间处理变得更加简单，它提供了不同的方法来操作日期和时间。

在本教程中，我们将探讨如何使用日期时间API和Calendar类获取一年的开始日期和结束日期。

## 2. 使用日期时间API

日期时间API中的[LocalDate](https://www.baeldung.com/java-creating-localdate-with-values)和[TemporalAdjuster](https://www.baeldung.com/java-temporal-adjuster#:~:text=TemporalAdjuster%20allows%20us%20to%20perform,util.)类可以轻松获取一年的开始日期和结束日期。

下面是使用这些类的示例：

```java
@Test
void givenCurrentDate_whenGettingFirstAndLastDayOfYear_thenCorrectDatesReturned() {
    LocalDate today = LocalDate.now();
    LocalDate firstDay = today.with(firstDayOfYear());
    LocalDate lastDay = today.with(lastDayOfYear());

    assertEquals("2023-01-01", firstDay.toString());
    assertEquals("2023-12-31", lastDay.toString());
}
```

首先，我们创建一个LocalDate对象来获取当前日期。接下来，我们通过对当前日期对应的对象调用with()和firstDayOfYear()来获取一年中的第一天。

此外，我们在today上调用with()和lastDayOfYear()来获取一年的最后一天。

值得注意的是，firstDayOfYear()和lastDayOfYear()是TemporalAdjuster类的静态方法。

最后，我们断言一年的第一天和最后一天等于预期结果。

## 3. 使用Calendar和Date类

传统的[Calendar](https://www.baeldung.com/java-get-week-number)和[Date](https://www.baeldung.com/java-date-to-localdate-and-localdatetime)类也可以获取一年的开始和结束日期。

### 3.1 获取一年的开始

让我们使用Calendar和Date来获取一年的开始：

```java
@Test
void givenCalendarWithSpecificDate_whenFormattingToISO8601_thenFormattedDateMatches() {
    Calendar cal = Calendar.getInstance();
    cal.set(Calendar.YEAR, 2023);
    cal.set(Calendar.DAY_OF_YEAR, 1);
    Date firstDay = cal.getTime();
        
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    String formattedDate = sdf.format(firstDay);
    
    assertEquals("2023-01-01", formattedDate);
}
```

这里，我们创建了一个新的Calendar实例，并设置了年份和日期。然后，我们获取Date对象并将其格式化为预期的开始日期。

最后，我们断言返回的日期等于预期的日期。

### 3.2 获取一年的结束

类似地，获取最后一天的方法如下：

```java
@Test
void givenCalendarSetToFirstDayOfYear_whenFormattingDateToISO8601_thenFormattedDateMatchesLastDay() {
    Calendar cal = Calendar.getInstance();
    cal.set(Calendar.YEAR, 2023);
    cal.set(Calendar.MONTH, 11);
    cal.set(Calendar.DAY_OF_MONTH, 31);
    Date lastDay = cal.getTime();
        
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    String formattedDate = sdf.format(lastDay);
        
    assertEquals("2023-12-31", formattedDate);
}
```

在上面的代码中，我们设置了一年最后一天的年、月、日。此外，我们格式化了日期，并断言它等于预期日期。

## 4. 总结

在本文中，我们学习了如何使用现代Date Time API以及旧版Calendar和Date类来获取一年的开始和结束日期。与Calendar和Date相比，Date Time API提供了更简洁、更直观的API。