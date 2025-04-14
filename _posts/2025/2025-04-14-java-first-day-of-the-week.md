---
layout: post
title:  如何在Java中使用LocalDate确定一周第一天的日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在这个简短的教程中，我们将讨论如何使用Java中的[LocalDate](https://www.baeldung.com/java-8-date-time-intro)输入来查找一周的第一天。

## 2. 问题陈述

我们经常需要用一周的第一天来为业务逻辑建立一周的边界，例如为员工建立时间跟踪系统。

在Java 8之前，[JodaTime](https://www.baeldung.com/joda-time)库用于查找一周的第一天。然而，Java 8之后，不再支持该功能。因此，我们将了解如何使用java.time.LocalDate类提供的功能来查找一周的第一天。

## 3. Calendar

我们可以使用[java.util.Calendar](https://www.baeldung.com/java-gregorian-calendar)类来选择一周中的某一天，并进行时间回溯。首先，我们可以根据第一天(星期日/星期一)的定义，循环到一周的开始。

让我们设置相同的Calendar类的对象：

```java
Calendar calendar = Calendar.getInstance();
ZoneId zoneId = ZoneId.systemDefault();
Date date = Date.from(localDate.atStartOfDay(zoneId).toInstant());
calendar.setTime(date);
```

设置Calendar对象后，我们必须确定一个固定的日期作为一周的第一天。根据ISO标准，可以是星期一，也可以是星期日，就像世界上许多国家(例如美国)所遵循的那样。我们可以不断循环遍历，直到找到我们确定的一周的第一天：

```java
while (calendar.get(Calendar.DAY_OF_WEEK) != Calendar.MONDAY) {
    calendar.add(Calendar.DATE, -1);
}
```

我们可以看到，每次减去一天直到星期一，就能得到一周第一天的日期。[Calendar.MONDAY](https://www.baeldung.com/java-creating-localdate-with-values)是Calendar类中定义的常量，现在我们可以将日历日期转换为[java.time.LocalDate](https://www.baeldung.com/java-creating-localdate-with-values)：

```java
LocalDateTime.ofInstant(calendar.toInstant(), calendar.getTimeZone().toZoneId()).toLocalDate()
```

## 4. TemporalAdjuster

[TemporalAdjuster](https://www.baeldung.com/java-temporal-adjuster)允许我们执行复杂的日期操作，例如，我们可以获取下一个星期日的日期、当前月份的最后一天或下一年的第一天。

根据我们确定一周第一天的方式，我们可以使用它来确定一周中的星期一或星期日的日期：

```java
DayOfWeek weekStart = DayOfWeek.MONDAY;
return localDate.with(TemporalAdjusters.previousOrSame(weekStart));
```

previousOrSame函数返回一个TemporalAdjuster对象，TemporalAdjuster对象返回指定周的上一个日期，如果当前日期已经是该日期，则返回该日期。我们可以使用它来调整日期并计算给定日期对应的一周的起始日期。

## 5. TemporalField

TemporalField表示日期时间字段，例如月份或小时/分钟。我们可以调整输入日期，以获取给定日期对应的一周的第一天。

我们可以使用dayOfWeek函数根据WeekFields访问一周的第一天，Java日期和时间API的WeekFields类表示基于周的年份及其组成部分，包括周数、星期几和基于周的年份。

当一周的第一天是星期日时，星期几的编号从1到7，其中1表示星期日，7表示星期六。这为使用ISO星期日期提供了一种便捷的方法，可以帮助我们获取一周第一天的日期：

```java
TemporalField fieldISO = WeekFields.of(locale).dayOfWeek();
return localDate.with(fieldISO, 1);
```

在这种情况下，我们传递了语言环境；因此，一周的第一天是星期日还是星期一将取决于具体地区。为了避免这种情况，我们可以使用ISO标准，该标准接收星期一作为一周的第一天：

```java
TemporalField dayOfWeek = WeekFields.ISO.dayOfWeek();
return localDate.with(dayOfWeek, dayOfWeek.range().getMinimum());
```

该代码片段使用ISO日历系统返回给定LocalDate实例的一周第一天的日期，该系统以星期一作为一周的第一天，它通过将星期几字段设置为给定LocalDate实例的最小有效值(即，星期一为1)来实现此目的。

## 6. 总结

在本文中，我们从Java中的LocalDate中获取了一周第一天的日期。我们还学习了如何使用Calendar类来实现同样的功能，以及使用TemporalAdjuster和TemporalField的多种方法。