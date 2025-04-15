---
layout: post
title:  在Java中从日期中减去天数
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将探索在Java中从Date对象中减去天数的各种方法。

我们将首先使用Java 8中引入的[日期时间API](https://www.baeldung.com/java-8-date-time-intro)。然后，我们将学习如何使用java.util包中的类来实现这一点，最后，我们将借助[Joda-Time](https://www.baeldung.com/joda-time)库来实现同样的目的。

## 2. java.time.LocalDateTime

**Java 8中引入的Date/Time API是目前日期和时间计算最可行的选择**。

让我们看看如何从Java 8的java.util.LocalDateTime对象中减去天数：

```java
@Test
public void givenLocalDate_whenSubtractingFiveDays_dateIsChangedCorrectly() {
    LocalDateTime localDateTime = LocalDateTime.of(2022, 4, 20, 0, 0);

    localDateTime = localDateTime.minusDays(5);

    assertEquals(15, localDateTime.getDayOfMonth());
    assertEquals(4, localDateTime.getMonthValue());
    assertEquals(2022, localDateTime.getYear());
}
```

## 3. java.util.Calendar

java.util中的Date和Calendar是Java 8之前版本中最常用的日期操作实用程序类。

让我们使用java.util.Calendar从一个日期中减去5天：

```java
@Test
public void givenCalendarDate_whenSubtractingFiveDays_dateIsChangedCorrectly() {
    Calendar calendar = Calendar.getInstance();
    calendar.set(2022, Calendar.APRIL, 20);

    calendar.add(Calendar.DATE, -5);

    assertEquals(15, calendar.get(Calendar.DAY_OF_MONTH));
    assertEquals(Calendar.APRIL, calendar.get(Calendar.MONTH));
    assertEquals(2022, calendar.get(Calendar.YEAR));
}
```

**在使用时应该小心，因为它们存在一些设计缺陷并且不是线程安全的**。

可以在有关迁移到新的[Java 8日期时间API](https://baeldung.com/migrating-to-java-8-date-time-api)的文章中阅读有关与遗留代码的交互以及两个API之间的差异的更多信息。

## 4. Joda-Time

我们可以使用Joda-Time作为Java初始日期和时间处理解决方案的更好替代方案，该库提供了更直观的API、多种日历系统、线程安全性和不可变对象。

为了使用[Joda-Time](https://mvnrepository.com/artifact/joda-time/joda-time)，我们需要将其作为依赖包含在pom.xml文件中：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

从Joda-Time的DateTime对象中减去5天：

```java
@Test
public void givenJodaDateTime_whenSubtractingFiveDays_dateIsChangedCorrectly() {
    DateTime dateTime = new DateTime(2022, 4, 20, 12, 0, 0);

    dateTime = dateTime.minusDays(5);

    assertEquals(15, dateTime.getDayOfMonth());
    assertEquals(4, dateTime.getMonthOfYear());
    assertEquals(2022, dateTime.getYear());
}

```

Joda-Time对于遗留代码来说是一个很好的解决方案，**但是，该项目已正式“完成”，其作者建议迁移到Java 8的Date/Time API**。

## 5. 总结

在这篇短文中，我们探讨了从日期对象中减去天数的几种方法。