---
layout: post
title:  在Java中将日期转换为Unix时间戳
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在计算机科学中，Unix时间戳(也称为[纪元时间](https://www.baeldung.com/java-localdate-epoch#epoch-vs-localdate))是表示特定时间点的标准方式，它表示自1970年1月1日以来经过的秒数。

在本教程中，我们将讲解如何将经典日期转换为Unix时间戳。首先，我们将探索如何使用JDK内置方法来实现此目的。然后，我们将演示如何使用外部库(例如Joda-Time)来实现相同的目的。

## 2. 使用Java 8+日期时间API

Java 8引入了一个[新的Date-Time API](https://www.baeldung.com/java-8-date-time-intro)，我们可以用它来解答我们的核心问题。这个新的API附带了一些操作日期的方法和类，那么，让我们仔细看看每个选项。

### 2.1 使用Instant类

简而言之，[Instant](https://www.baeldung.com/java-instant-vs-localdatetime#1-using-instant)类模拟了时间轴上的瞬时点，**该类提供了一种直接简洁的方法，可以从给定日期获取Unix时间**。

那么，让我们看看它的实际效果：

```java
@Test
void givenDate_whenUsingInstantClass_thenConvertToUnixTimeStamp() {
    Instant givenDate = Instant.parse("2020-09-08T12:16:40Z");

    assertEquals(1599567400L, givenDate.getEpochSecond());
}
```

我们可以看到，Instant类提供了getEpochSecond()方法来获取指定日期的纪元时间戳(以秒为单位)。

### 2.2 使用LocalDateTime类

[LocalDateTime](https://www.baeldung.com/java-8-date-time-intro#3-working-with-localdatetime)是将日期转换为纪元时间时需要考虑的另一个选项，此类表示日期和时间的组合，通常以年、月、日、时、分、秒的形式显示。

**通常，此类提供toEpochSecond()方法来获取指定日期时间的纪元时间(以秒为单位)**：

```java
@Test
void givenDate_whenUsingLocalDateTimeClass_thenConvertToUnixTimeStamp() {
    LocalDateTime givenDate = LocalDateTime.of(2023, 10, 19, 22, 45);

    assertEquals(1697755500L, givenDate.toEpochSecond(ZoneOffset.UTC));
}
```

如上所示，与其他方法不同，toEpochSecond()接收一个[ZoneOffset](https://www.baeldung.com/java-zone-offset)对象，它允许我们定义时区UTC的固定偏移量。

## 3. 使用旧版日期API

或者，我们可以使用旧API中的Date和[Calendar](https://www.baeldung.com/java-gregorian-calendar)类来实现相同的效果。那么，让我们深入研究一下，看看如何在实践中使用它们。

### 3.1 使用Date类

在Java中，Date类以毫秒精度表示特定的时间点，**它提供了最简单的方法之一，即通过getTime()方法将日期转换为Unix时间戳**：

```java
@Test
void givenDate_whenUsingDateClass_thenConvertToUnixTimeStamp() throws ParseException {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    dateFormat.setTimeZone(TimeZone.getTimeZone("UTC"));
    Date givenDate = dateFormat.parse("2023-10-15 22:00:00");

    assertEquals(1697407200L, givenDate.getTime() / 1000);
}
```

通常，该方法返回自传入日期纪元以来的毫秒数。如我们所见，我们将结果除以1000，得到了纪元的秒数。**然而，此类已过时，不应在处理日期时使用**。

### 3.2 使用Calendar类

类似地，我们可以使用同一个包java.util中的Calendar类，此类提供了许多设置和操作日期的方法。

**使用Calendar，我们必须调用getTimeInMillis()来返回指定日期的Unix时间**：

```java
@Test
void givenDate_whenUsingCalendarClass_thenConvertToUnixTimeStamp() throws ParseException {
    Calendar calendar = new GregorianCalendar(2023, Calendar.OCTOBER, 17);
    calendar.setTimeZone(TimeZone.getTimeZone("UTC"));

    assertEquals(1697500800L, calendar.getTimeInMillis() / 1000);
}
```

请注意，此方法顾名思义，以毫秒为单位返回时间戳。**这种选择的缺点是，Calendar已被声明为事实上的遗留API，因为它属于旧API**。

## 4. 使用Joda-Time

另一个解决方案是使用[Joda-Time](https://www.baeldung.com/java-gregorian-calendar)库，在开始使用该库之前，让我们先将其[依赖](https://mvnrepository.com/artifact/joda-time/joda-time)添加到pom.xml中：

```xml
<dependency>
    <groupId>joda-time</groupId> 
    <artifactId>joda-time</artifactId> 
    <version>2.12.6</version> 
</dependency>
```

**Joda-Time提供了Instant类的版本，我们可以用它来应对我们的挑战**。那么，让我们用一个新的测试用例来说明如何使用这个类：

```java
@Test
void givenDate_whenUsingJodaTimeInstantClass_thenConvertToUnixTimeStamp() {
    org.joda.time.Instant givenDate = org.joda.time.Instant.parse("2020-09-08T12:16:40Z");

    assertEquals(1599567400L, givenDate.getMillis() / 1000);
}
```

如图所示，Instant类提供了一种直接的方法来获取自纪元以来的毫秒数。

**DateTime类是使用Joda-Time时要考虑的另一种解决方案**，它提供了getMillis()方法，返回自DateTime时刻以来经过的毫秒数：

```java
@Test
void givenDate_whenUsingJodaTimeDateTimeClass_thenConvertToUnixTimeStamp() {
    DateTime givenDate = new DateTime("2020-09-08T12:16:40Z");

    assertEquals(1599567400L, givenDate.getMillis() / 1000);
}
```

测试用例成功通过。

## 5. 总结

在这篇短文中，我们探讨了将给定日期转换为Unix时间戳的不同方法。

首先，我们解释了如何使用核心JDK方法和类来实现这一点。然后，我们展示了如何使用Joda-Time实现同样的目标。