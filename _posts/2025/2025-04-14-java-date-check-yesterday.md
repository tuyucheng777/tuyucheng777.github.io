---
layout: post
title:  检查日期对象是否等于昨天
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在Java应用程序中处理日期和时间数据时，出于各种目的(例如安排任务、提醒或报告)比较日期通常至关重要，一种常见的情况是需要确定给定日期相对于当前日期是否对应于昨天。在本教程中，我们将探索各种方法来确定给定日期对象是否属于昨天。

## 2. 使用java.util.Calendar

**一种常见的方法是使用java.util.Calendar类来操作日期和时间信息**，为了比较昨天的日期，我们通过Calendar.getInstance()实例化一个Calendar对象。接下来，我们使用calendar.setTime(new Date())将其时间设置为当前日期，然后使用calendar.add(Calendar.DATE, -1)将其减去一天，这样就得到了昨天的日期。

以下是演示这些概念的代码片段：

```java
Date date = new Date(System.currentTimeMillis() - (1000 * 60 * 60 * 24));
Calendar expectedCalendar = Calendar.getInstance();
expectedCalendar.setTime(date);

Calendar actualCalendar = Calendar.getInstance();
actualCalendar.add(Calendar.DATE, -1);

boolean isEqualToYesterday = expectedCalendar.get(Calendar.YEAR) == actualCalendar.get(Calendar.YEAR) &&
    expectedCalendar.get(Calendar.MONTH) == actualCalendar.get(Calendar.MONTH) &&
    expectedCalendar.get(Calendar.DAY_OF_MONTH) == actualCalendar.get(Calendar.DAY_OF_MONTH);

assertTrue(isEqualToYesterday);
```

**使用此方法直接比较日期可能会失败，因为两个日期之间存在毫秒差异**。这是因为Date对象同时捕获日期和时间，包括毫秒。如果时间部分不完全相同，比较可能会产生不正确的结果。

为了缓解此问题，一种方法是在进行比较之前截断两个日期的时间部分。我们使用get(Calendar.YEAR)、get(Calendar.MONTH)和get(Calendar.DAY_OF_MONTH)方法分别从两个日期中提取年、月、日部分，以单独判断actualCalendar是否等于昨天。

**尽管这些类被视为遗留API，但它们仍然被广泛使用，尤其是在没有较新Java版本的环境中**。

## 3. 使用java.util.Date毫秒

这种方法利用了Date对象内部存储自纪元(UTC时间1970年1月1日 00:00:00)以来的毫秒数这一特性，**它需要计算昨天午夜的毫秒数，并将其与Date对象的getTime()值进行比较**。

首先，我们使用ZonedDateTime计算系统默认时区中昨天(午夜)开始的时间戳(以毫秒为单位)。具体方法是：从当前日期和时间中减去一天，然后截断到午夜，最终获取时间戳。

随后，我们将此计算值与给定expectedDate的getTime()值进行比较。**通过验证expectedDate是否在从昨天午夜时间戳到下一个午夜(yesterdayMidnightMillis + 86400000毫秒)的范围内，我们确定yesterdayMidnightMillis是否对应于昨天的日期**。

让我们用一个简单的例子来演示这种方法：

```java
Date expectedDate = new Date(System.currentTimeMillis() - (1000 * 60 * 60 * 24));
ZonedDateTime yesterdayMidnight = ZonedDateTime.now().minusDays(1).truncatedTo(ChronoUnit.DAYS);
long yesterdayMidnightMillis = yesterdayMidnight.toInstant().toEpochMilli();

boolean isEqualToYesterday = expectedDate.getTime() >= yesterdayMidnightMillis && expectedDate.getTime() < yesterdayMidnightMillis + 86_400_000;

assertTrue(isEqualToYesterday);
```

此方法提供了一种简单高效的毫秒级日期比较方法，它适用于需要直接与旧版Date对象进行比较的场景。

## 4. 使用java.time.LocalDate 

**处理日期操作的另一种方法是使用LocalDate类及其minusDays()方法**，此方法是Java 8中引入的现代[日期和时间API](https://www.baeldung.com/java-8-date-time-intro)的一部分。

LocalDate提供了一种更直接、更直观的日期处理方式，**它专注于年、月、日部分，不包含时间部分**，这使得它非常适合仅处理日期的操作，例如记录历史日期或过滤报告日期。

要使用此方法比较昨天的日期，我们首先使用LocalDate.now()获取当前日期。然后，我们只需在LocalDate对象上调用minusDays(1)方法减去一天，即可得到昨天的日期：

```java
Date date = new Date(System.currentTimeMillis() - (1000 * 60 * 60 * 24));
LocalDate expectedLocalDate = LocalDate.of(date.getYear() + 1900, date.getMonth() + 1, date.getDate());

LocalDate actualLocalDate = LocalDate.now().minusDays(1);
boolean isEqualToYesterday = expectedLocalDate.equals(actualLocalDate);
assertTrue(isEqualToYesterday);
```

在此示例中，我们通过从Date对象中提取年、月、日部分来指定昨天的日期。**但请注意，getYear()方法返回自1900年以来的年份，而getMonth()返回从0开始的月份索引**。因此，我们将1900添加到getYear()，并将1添加到getMonth()，以将它们调整为标准日期表示。

**对于Java 8及更高版本，建议使用LocalDate进行日期操作**。

## 5. 使用Joda-Time

[Joda-Time](https://www.baeldung.com/joda-time)是一个流行的Java日期和时间库，我们可以用这个库来检查一个日期对象是否代表昨天的日期。**为了使用Joda-Time比较昨天的日期，我们使用DateTime类及其minusDays()方法**。

首先，我们使用DateTime.now()获取当前日期和时间。然后，我们通过调用minusDays(1)方法从当前日期中减去一天，得到昨天的日期：

```java
Date date = new Date(System.currentTimeMillis() - (1000 * 60 * 60 * 24));
DateTime expectedDateTime = new DateTime(date).withTimeAtStartOfDay();

DateTime actualDateTime = DateTime.now().minusDays(1).withTimeAtStartOfDay();

boolean isEqualToYesterday = expectedDateTime.equals(actualDateTime);

assertTrue(isEqualToYesterday);
```

与Calendar类类似，Joda-Time的DateTime类会随日期一起捕获时间，因此直接比较它们可能会因毫秒级的差异而导致不准确。

为了解决这个问题，**我们可以利用Joda-Time提供的withTimeAtStartOfDay()方法，而不是将日期值分解为年、月、日三个部分**。此方法将DateTime对象的时间部分设置为一天的开始，相当于将时间重置为午夜(00:00:00)。通过将withTimeAtStartOfDay()应用于DateTime对象，我们可以确保只有日期部分在比较时仍然有效。

## 6. 总结

在本文中，我们探讨了几种判断日期对象是否为昨天日期的方法，对于大多数使用Java 8或更高版本的现代Java应用程序，通常推荐使用LocalDate方法，因为它清晰、不可变，并且支持各种日期操作。

虽然Joda-Time是一个成熟且稳定的库，但随着越来越多的项目转向标准Java日期和时间API，其未来的开发和支持可能会变得越来越不确定。