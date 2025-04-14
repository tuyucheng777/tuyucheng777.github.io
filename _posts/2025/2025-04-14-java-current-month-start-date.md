---
layout: post
title:  在Java中获取当前月份的第一天
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在这篇短文中，我们将重点介绍如何在Java中获取当前月份的第一天。

首先，我们将阐明如何使用JDK解决方案来实现这一点。然后，我们将展示如何使用Joda Time等第三方库实现相同的效果。

## 2. 问题介绍

与往常一样，让我们通过一个例子来理解主要问题，假设当前日期是2023-10-15：

```java
LocalDate currentDate = LocalDate.of(2023, 10, 15);
```

这里我们的目标是获取10月的第一天，即2023年10月1日。

## 3. 使用Calendar类

[Calendar](https://www.baeldung.com/java-gregorian-calendar)类提供了一组便捷的方法来操作日期和时间，它提供了一种便捷的方法来获取给定月份的第一天。

那么，让我们看看它的实际效果：

```java
@Test
void givenMonth_whenUsingCalendar_thenReturnFirstDay() {
    Date currentDate = new GregorianCalendar(2023, Calendar.NOVEMBER, 23).getTime();
    Date expectedDate = new GregorianCalendar(2023, Calendar.NOVEMBER, 1).getTime();

    Calendar cal = Calendar.getInstance();
    cal.setTime(currentDate);
    cal.set(Calendar.DAY_OF_MONTH, 1);

    assertEquals(expectedDate, cal.getTime());
}
```

**如我们所见，我们将日历时间更改为给定的日期，然后，我们调用set(Calendar.DAY_OF_MONTH, 1)将当前月份的日期设置为1**。

此外，我们使用getTime()方法返回新日期，该日期表示该月的第一天。

## 4. 使用Java 8日期时间API

Java 8附带了许多现成的功能，例如[日期时间API](https://www.baeldung.com/java-8-date-time-intro)，**引入这个新API是为了解决基于Calendar类的旧API的缺点**。

请注意，当涉及处理日期时，这个新的Java 8 API是最好的方法。

### 4.1 使用LocalDate类

如果我们想获取当前月份的第一天，[LocalDate](https://www.baeldung.com/java-creating-localdate-with-values)是一个不错的选择；**该类提供了withDayOfMonth()方法，我们可以使用它来返回修改后的新日期**。

那么，让我们在实践中看看：

```java
@Test
void givenMonth_whenUsingLocalDate_thenReturnFirstDay() {
    LocalDate currentDate = LocalDate.of(2023, 9, 6);
    LocalDate expectedDate = LocalDate.of(2023, 9, 1);

    assertEquals(expectedDate, currentDate.withDayOfMonth(1));
}
```

如上所示，我们使用withDayOfMonth(1)将指定日期的日期设置为值1。这样，我们就得到了该月的第一天。

### 4.2 使用TemporalAdjusters类

类似地，我们可以使用[TemporalAdjusters](https://www.baeldung.com/java-temporal-adjuster)来。**顾名思义，此类代表了几个调整器，旨在修改时间对象，例如天和月**。

在这些调整器中，我们可以看到firstDayOfMonth。接下来，让我们添加一个新的测试用例来测试调整器：

```java
@Test
void givenMonth_whenUsingTemporalAdjusters_thenReturnFirstDay() {
    LocalDate currentDate = LocalDate.of(2023, 7, 19);
    LocalDate expectedDate = LocalDate.of(2023, 7, 1);

    assertEquals(expectedDate, currentDate.with(TemporalAdjusters.firstDayOfMonth()));
}
```

通常，firstDayOfMonth返回特定月份的第一天。请注意，我们使用了with()方法来获取调整后的新日期副本。

### 4.3 使用YearMonth类

YearMonth引入了atDay()方法，将特定的年份/月份与作为参数传递的日期结合起来。

现在，让我们看看如何使用这种方法来实现我们的目标：

```java
@Test
void givenMonth_whenUsingYearMonth_thenReturnFirstDay() {
    YearMonth currentDate = YearMonth.of(2023, 4);
    LocalDate expectedDate = LocalDate.of(2023, 4, 1);

    assertEquals(expectedDate, currentDate.atDay(1));
}
```

**这里，我们指定1作为参数来表示给定年份和月份的第一天**。

## 5. 使用Joda Time库

另一个解决方案是使用[Joda Time库](https://www.baeldung.com/joda-time)，它提供了Java 8之前用于日期时间操作的标准API。

在开始使用这个库之前，我们首先需要将它的[依赖](https://mvnrepository.com/artifact/joda-time/joda-time)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

**Joda Time附带了LocalDate类的一个变体来表示日期**，让我们用一个新的测试用例来演示这个类的用法：

```java
@Test
void givenMonth_whenUsingJodaTime_thenReturnFirstDay() {
    org.joda.time.LocalDate currentDate = org.joda.time.LocalDate.parse("2023-5-10");
    org.joda.time.LocalDate expectedDate = org.joda.time.LocalDate.parse("2023-5-1");

    assertEquals(expectedDate, currentDate.dayOfMonth()
        .withMinimumValue());
}
```

顾名思义，dayOfMonth()可以检索给定月份的日期。此外，我们使用withMinimumValue()方法将返回日期的日期设置为最小值1。

## 6. 总结

在这个简短的教程中，我们探索了使用Java获取当前月份第一天的不同方法。

首先，我们讨论了如何使用JDK类来实现这一点，然后，我们演示了如何使用Joda Time API实现同样的目标。
