---
layout: post
title:  在Java中将Long转换为Date
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在Java中处理日期时，我们经常会看到以Long值表示的日期/时间值，表示自[纪元](https://www.baeldung.com/java-localdate-epoch#epoch-vs-localdate)1970年1月1日00:00:00 GMT以来的天数、秒数或毫秒数。

在本简短教程中，我们将探索在Java中将Long值转换为日期的不同方法。首先，我们将解释如何使用核心JDK类来实现此操作。然后，我们将展示如何使用第三方Joda-Time库实现相同的目的。

## 2. 使用Java 8+日期时间API

Java 8因其为Java领域带来的全新日期时间API特性而备受赞誉，引入此API主要是为了弥补旧日期API的不足。那么，让我们仔细研究一下此API的功能，以解答我们的核心问题。

### 2.1 使用Instant类

最简单的解决方案是使用[Java 8新日期时间API](https://www.baeldung.com/java-8-date-time-intro)中引入的[Instant](https://www.baeldung.com/java-instant-vs-localdatetime#1-using-instant)类，**此类描述时间轴上的单个瞬时点**。

那么，让我们在实践中看看：

```java
@Test
void givenLongValue_whenUsingInstantClass_thenConvert() {
    Instant expectedDate = Instant.parse("2020-09-08T12:16:40Z");
    long seconds = 1599567400L;

    Instant date = Instant.ofEpochSecond(seconds);

    assertEquals(expectedDate, date);
}
```

如上所示，我们使用了ofEpochSecond()方法创建了Instant类的对象。请记住，我们也可以使用ofEpochMilli()方法以毫秒为单位创建Instant实例。

### 2.2 使用LocalDate类

[LocalDate](https://www.baeldung.com/java-creating-localdate-with-values)是将Long值转换为日期时可以考虑的另一个选项，该类模拟的是经典日期类型，例如2023-10-17，但不包含时间细节。

通常，我们可以使用LocalDate#ofEpochDay方法来实现我们的目的：

```java
@Test
void givenLongValue_whenUsingLocalDateClass_thenConvert() {
    LocalDate expectedDate = LocalDate.of(2023, 10, 17);
    long epochDay = 19647L;

    LocalDate date = LocalDate.ofEpochDay(epochDay);

    assertEquals(expectedDate, date);
}
```

ofEpochDay()方法根据给定的纪元日期创建LocalDate类的实例。

## 3. 使用旧版日期API

在Java 8之前，我们通常使用java.util包中的Date或Calendar类来实现目标。那么，让我们看看如何使用这两个类将Long值转换为日期。

### 3.1 使用Date类

Date类以毫秒为精度表示特定的时间点，顾名思义，它包含许多可用于操作日期的方法。**它提供了将Long值转换为日期的最简单方法，因为它提供了一个接收Long类型参数的重载构造函数**。

那么，让我们看看它的实际效果：

```java
@Test
void givenLongValue_whenUsingDateClass_thenConvert() {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    dateFormat.setTimeZone(TimeZone.getTimeZone("UTC"));
    Date expectedDate = dateFormat.parse("2023-10-15 22:00:00");
    long milliseconds = 1689458400000L;

    Date date = new Date(milliseconds);

    assertEquals(expectedDate, date);
}
```

请注意，Date类已经过时，属于旧API。因此，它并非处理日期的最佳方法。

### 3.2 使用Calendar类

另一个解决方案是使用旧日期API中的[Calendar](https://www.baeldung.com/java-gregorian-calendar)类，**该类提供了setTimeInMillis(long value)方法，我们可以使用该方法将时间设置为给定的Long值**。

现在，让我们使用另一个测试用例来举例说明该方法的使用：

```java
@Test
void givenLongValue_whenUsingCalendarClass_thenConvert() {
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    dateFormat.setTimeZone(TimeZone.getTimeZone("UTC"));
    Date expectedDate = dateFormat.parse("2023-07-15 22:00:00");
    long milliseconds = 1689458400000L;

    Calendar calendar = Calendar.getInstance();
    calendar.setTimeZone(TimeZone.getTimeZone("UTC"));
    calendar.setTimeInMillis(milliseconds);

    assertEquals(expectedDate, calendar.getTime());
}
```

类似地，指定的Long值表示自纪元以来经过的毫秒数。

## 4. 使用Joda-Time

最后，我们可以使用[Joda-Time库](https://www.baeldung.com/joda-time)来应对挑战。首先，让我们将其[依赖](https://mvnrepository.com/artifact/joda-time/joda-time)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

类似地，Joda-Time也提供了LocalDate类的版本。那么，让我们看看如何使用它将Long值转换为LocalDate对象：

```java
@Test
void givenLongValue_whenUsingJodaTimeLocalDateClass_thenConvert() {
    org.joda.time.LocalDate expectedDate = new org.joda.time.LocalDate(2023, 7, 15);
    long milliseconds = 1689458400000L;

    org.joda.time.LocalDate date = new org.joda.time.LocalDate(milliseconds, DateTimeZone.UTC);

    assertEquals(expectedDate, date);
}
```

如图所示，LocalDate提供了一种从Long值构造日期的直接方法。

## 5. 总结

在这篇短文中，我们详细解释了如何在Java中将Long值转换为日期。

首先，我们了解了如何使用JDK内置类进行转换。然后，我们演示了如何使用Joda-Time库实现相同的目标。