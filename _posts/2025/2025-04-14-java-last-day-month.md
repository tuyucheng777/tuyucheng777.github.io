---
layout: post
title:  如何在Java中获取月份的最后一天
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在这个简短的教程中，我们将讲解如何在Java中获取给定月份的最后一天。

首先，我们将重点介绍如何使用核心Java方法；然后，我们将说明如何使用Joda Time库实现相同的目标。

## 2. Java 8之前

在Java 8之前，Date和Calendar类是用于在Java中操作时间和日期的良好选择。

**通常，Calendar提供了一组方法和常量，我们可以使用这些方法和常量来访问和操作时间信息，例如天、月和年**。

那么，让我们看看如何使用它来获取特定月份的最后一天：

```java
static int getLastDayOfMonthUsingCalendar(int month) {
    Calendar cal = Calendar.getInstance();
    cal.set(Calendar.MONTH, month);
    return cal.getActualMaximum(Calendar.DAY_OF_MONTH);
}
```

如我们所见，我们使用Calendar.getInstance()创建了一个Calendar对象。然后，我们将日历月份设置为指定的月份。此外，我们使用getActualMaximum()方法和Calendar.DAY_OF_MONTH字段来获取该月的最后一天。

现在，让我们添加一个测试用例来确认是否正确：

```java
@Test
void givenMonth_whenUsingCalendar_thenReturnLastDay() {
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingCalendar(0));
    assertEquals(30, LastDayOfMonth.getLastDayOfMonthUsingCalendar(3));
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingCalendar(9));
}
```

**这里需要强调的一点是，月份值是从0开始的**。例如，一月的值为0，因此我们需要从所需的月份值中减去1。

## 3. Java 8日期时间API

Java 8带来了许多新功能和增强功能，在这些功能中，包括[日期时间API](https://www.baeldung.com/java-8-date-time-intro)，**引入此API是为了克服基于Date和Calendar类的旧API的局限性**。

那么，让我们深入研究一下，看看这个新API提供了哪些选项来获取一个月的最后一天。

### 3.1 使用YearMonth

操作月份最常用的方法之一是使用YearMonth类，顾名思义，它表示年份和月份的组合。

**此类提供atEndOfMonth()方法，该方法返回一个LocalDate对象，该对象表示给定月份的结束日期**。

那么，让我们在实践中看看：

```java
static int getLastDayOfMonthUsingYearMonth(YearMonth date) {
    return date.atEndOfMonth()
        .getDayOfMonth();
}
```

简而言之，我们使用atEndOfMonth()获取该月的最后一天。然后，我们链接getDayOfMonth()来检索返回的LocalDate对象的日期。

与往常一样，让我们添加一个测试用例：

```java
@Test
void givenMonth_whenUsingYearMonth_thenReturnLastDay() {
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingYearMonth(YearMonth.of(2023, 1)));
    assertEquals(30, LastDayOfMonth.getLastDayOfMonthUsingYearMonth(YearMonth.of(2023, 4)));
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingYearMonth(YearMonth.of(2023, 10)));
}
```

### 3.2 使用TemporalAdjusters

另一个解决方案是使用[TemporalAdjusters](https://www.baeldung.com/java-temporal-adjuster)类。**通常，此类提供了几个现成的静态方法，我们可以使用这些方法来调整时间对象**。

那么，让我们看看如何使用它来获取LocalDate实例的最后一天：

```java
static int getLastDayOfMonthUsingTemporalAdjusters(LocalDate date) {
    return date.with(TemporalAdjusters.lastDayOfMonth())
        .getDayOfMonth();
}
```

如上所示，该类提供了调整器lastDayOfMonth()，它返回一个设置为该月最后一天的新日期。

此外，我们使用getDayOfMonth()方法来获取返回日期的日期。

接下来，使用测试用例来确认我们的方法：

```java
@Test
void givenMonth_whenUsingTemporalAdjusters_thenReturnLastDay() {
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingTemporalAdjusters(LocalDate.of(2023, 1, 1)));
    assertEquals(30, LastDayOfMonth.getLastDayOfMonthUsingTemporalAdjusters(LocalDate.of(2023, 4, 1)));
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingTemporalAdjusters(LocalDate.of(2023, 10, 1)));
}
```

## 4. Joda Time库

或者，我们可以使用[Joda Time API](https://www.baeldung.com/joda-time)来实现同样的目的。在Java 8之前，它是日期时间操作的事实标准。首先，我们需要将[Joda Time依赖](https://mvnrepository.com/artifact/joda-time/joda-time)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

类似地，Joda Time提供了LocalDate类来表示日期，**它包含withMaximumValue()方法，可用于获取月份的最后一天**。

让我们看看实际效果：

```java
static int getLastDayOfMonthUsingJodaTime(org.joda.time.LocalDate date) {
    return date.dayOfMonth()
        .withMaximumValue()
        .getDayOfMonth();
}
```

最后，让我们创建另一个测试用例：

```java
@Test
void givenMonth_whenUsingJodaTime_thenReturnLastDay() {
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingJodaTime(org.joda.time.LocalDate.parse("2023-1-1")));
    assertEquals(30, LastDayOfMonth.getLastDayOfMonthUsingJodaTime(org.joda.time.LocalDate.parse("2023-4-1")));
    assertEquals(31, LastDayOfMonth.getLastDayOfMonthUsingJodaTime(org.joda.time.LocalDate.parse("2023-10-1")));
}
```

测试成功通过。

## 5. 总结

在本文中，我们探讨了在Java中获取指定月份最后一天的不同方法。在此过程中，我们解释了如何使用核心Java来实现；然后，我们演示了如何使用Joda Time等第三方库。