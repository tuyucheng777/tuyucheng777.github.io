---
layout: post
title:  在Java中将当前日期增加一个月
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在这个简短的教程中，我们将学习如何在Java中将当前日期增加一个月。

首先，我们将了解如何使用核心Java方法实现这一点。然后，我们将了解如何使用外部库(例如Joda-Time和Apache Commons Lang3)实现同样的功能。

## 2. 核心Java方法

Java提供了几种便捷的方法来操作日期和时间，让我们探索一下在当前日期上增加一个月的不同方法。

### 2.1 使用Calendar类

对于Java 8之前的版本，我们可以使用Calendar来处理时间数据，该类提供了一组可用于操作日期和时间的方法。

那么，让我们看看它的实际效果：

```java
@Test
void givenCalendarDate_whenAddingOneMonth_thenDateIsChangedCorrectly() {
    Calendar calendar = Calendar.getInstance();
    // Dummy current date
    calendar.set(2023, Calendar.APRIL, 20);

    // add one month
    calendar.add(Calendar.MONTH, 1);

    assertEquals(Calendar.MAY, calendar.get(Calendar.MONTH));
    assertEquals(20, calendar.get(Calendar.DAY_OF_MONTH));
    assertEquals(2023, calendar.get(Calendar.YEAR));
}
```

**如我们所见，我们使用了add()方法将给定日期加一个月，Calendar.MONTH是表示月份的常量**。

### 2.2 使用Date类

如果我们想更改特定日期的月份，Date类是另一个值得考虑的选择。**然而，这个选择的缺点是该类已被弃用**。

让我们通过测试用例来看一下Date的用法：

```java
@SuppressWarnings("deprecation")
@Test
void givenDate_whenAddingOneMonth_thenDateIsChangedCorrectly() {
    Date currentDate = new Date(2023, Calendar.DECEMBER, 20);
    Date expectedDate = new Date(2024, Calendar.JANUARY, 20);

    currentDate.setMonth(currentDate.getMonth() + 1);

    assertEquals(expectedDate, currentDate);
}
```

如上所示，**Date类提供了getMonth()方法，该方法返回一个代表月份的数字**。此外，我们将返回的数字加1。然后，我们调用setMonth()来用新月份更新Date对象。

值得注意的是，始终建议使用Java 8的[新日期/时间API](https://www.baeldung.com/java-8-date-time-intro)，而不是旧的API。

### 2.3 使用LocalDate类

类似地，我们可以使用Java 8中引入的LocalDate类，**该类提供了一种直接简洁的方法，通过plusMonths()方法将月份相加到特定日期**：

```java
@Test
void givenJavaLocalDate_whenAddingOneMonth_thenDateIsChangedCorrectly() {
    LocalDate localDate = LocalDate.of(2023, 12, 20);

    localDate = localDate.plusMonths(1);

    assertEquals(1, localDate.getMonthValue());
    assertEquals(20, localDate.getDayOfMonth());
    assertEquals(2024, localDate.getYear());
}
```

## 3. 使用Joda-Time

如果Java 8不是一个选项，我们可以选择[Joda-Time库](https://www.baeldung.com/joda-time)来实现相同的目标。

首先，我们需要在pom.xml文件中添加它的[依赖](https://mvnrepository.com/artifact/joda-time/joda-time)：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10</version>
</dependency>
```

Joda-Time提供了LocalDate类的版本，那么，让我们看看如何使用它来增加一个月：

```java
@Test
void givenJodaTimeLocalDate_whenAddingOneMonth_thenDateIsChangedCorrectly() {
    org.joda.time.LocalDate localDate = new org.joda.time.LocalDate(2023, 12, 20);

    localDate = localDate.plusMonths(1);

    assertEquals(1, localDate.getMonthOfYear());
    assertEquals(20, localDate.getDayOfMonth());
    assertEquals(2024, localDate.getYear());
}
```

我们可以看到，LocalDate也带有相同的方法plusMonths()，**顾名思义，它允许我们增加一定数量的月份，在本例中为1**。

## 4. 使用Apache Commons Lang3

或者，我们可以使用[Apache Commons Lang3](https://www.baeldung.com/java-commons-lang-3)库；像往常一样，要开始使用这个库，我们首先需要添加[Maven依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3/)：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

通常，Apache Commons Lang3提供DateUtils实用程序类来执行一系列日期操作。

让我们通过一个实际的例子来看一下如何使用它：

```java
@Test
void givenApacheCommonsLangDateUtils_whenAddingOneMonth_thenDateIsChangedCorrectly() {
    Date currentDate = new GregorianCalendar(2023, Calendar.APRIL, 20, 4, 0).getTime();
    Date expectedDate = new GregorianCalendar(2023, Calendar.MAY, 20, 4, 0).getTime();

    assertEquals(expectedDate, DateUtils.addMonths(currentDate, 1));
}
```

我们使用了addMonths()方法将指定的月份加一，**这里需要注意的一点是，此方法返回一个新的Date对象。原始对象保持不变**。

## 5. 总结

在这篇短文中，我们探讨了在Java中向当前日期增加一个月的不同方法。

首先，我们了解了如何使用核心Java类来实现这一点。然后，我们学习了如何使用第三方库(例如Joda Time和Apache Commons Lang3)来实现同样的功能。