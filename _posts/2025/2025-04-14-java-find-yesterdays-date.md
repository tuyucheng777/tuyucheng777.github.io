---
layout: post
title:  在Java中获取昨天的日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在这个简短的教程中，我们将探索在Java中获取昨天日期的不同方法。

首先，我们将解释如何使用核心Java实现这一点。然后，我们将演示如何使用外部库(例如Joda-Time和Apache Commons Lang)来解决我们的主要难题。

## 2. Java 8之前

在Java 8之前，我们通常使用Date或Calendar来处理和操作日期/时间信息。那么，让我们看看如何使用这两个类来获取昨天的日期。

### 2.1 使用Date

Date类表示特定的时间点，它提供了一组用于操作和检索日期信息的方法。然而，需要指出的是，**此类已过时，并被标记为已弃用**。

那么，让我们看看实际效果：

```java
@Test
void givenDate_whenUsingDateClass_thenReturnYesterday() {
    Date date = new Date(2023, Calendar.DECEMBER, 20);
    Date yesterdayDate = new Date(date.getTime() - 24 * 60 * 60 * 1000);
    Date expectedYesterdayDate = new Date(2023, Calendar.DECEMBER, 19);

    assertEquals(expectedYesterdayDate, yesterdayDate);
}
```

我们使用了getTime()方法获取给定日期的毫秒数，然后，**我们减去一天，即24 * 60 * 60 * 1000毫秒**。

### 2.2 使用Calendar

如果我们想使用旧API，Calendar是另一个值得考虑的选择，该类提供了一组方法来管理时间数据，例如日期和月份。

例如，我们可以使用add()方法增加一定数量的天数。由于我们**想要获取昨天的日期，因此需要指定-1作为值**。

那么，让我们在实践中看看：

```java
@Test
void givenDate_whenUsingCalendarClass_thenReturnYesterday() {
    Calendar date = new GregorianCalendar(2023, Calendar.APRIL, 20, 4, 0);
    date.add(Calendar.DATE, -1);
    Calendar expectedYesterdayDate = new GregorianCalendar(2023, Calendar.APRIL, 19, 4, 0);

    assertEquals(expectedYesterdayDate, date);
}
```

正如预期的那样，测试成功通过。

## 3. Java 8日期时间API

Java 8因其新的[日期时间功能](https://www.baeldung.com/java-8-date-time-intro)而备受赞誉，该API包含一系列现成的类和方法，能够以更简洁、更人性化的方式计算日期和时间。那么，让我们仔细看看如何使用新的日期时间API获取昨天的日期。

### 3.1 使用LocalDate

最简单的解决方案之一是使用LocalDate类，**它提供了minusDays()方法，可以从给定日期中减去特定的天数。在本例中，我们需要减去1天**。

现在，让我们通过测试用例来举例说明LocalDate.minusDays()的使用：

```java
@Test
void givenDate_whenUsingLocalDateClass_thenReturnYesterday() {
    LocalDate localDate = LocalDate.of(2023, 12, 20);
    LocalDate yesterdayDate = localDate.minusDays(1);
    LocalDate expectedYesterdayDate = LocalDate.of(2023, 12, 19);

    assertEquals(expectedYesterdayDate, yesterdayDate);
}
```

如上所示，minusDays(1)返回一个表示昨天的新LocalDate对象。

### 3.2 使用Instant

另一个解决方案是使用Instant类，顾名思义，它模拟时间轴上的特定时间点。**通常，Instant类带有minus()方法，我们可以使用它来减去特定数量的毫秒数**。

那么，让我们用一个实际的例子来说明如何使用它来获取昨天的日期：

```java
@Test
void givenDate_whenUsingInstantClass_thenReturnYesterday() {
    Instant date = Instant.parse("2023-10-25");
    Instant yesterdayDate = date.minus(24 * 60 * 60 * 1000);
    Instant expectedYesterdayDate = Instant.parse("2023-10-24");

    assertEquals(expectedYesterdayDate, yesterdayDate);
}
```

正如我们之前提到的，24 * 60 * 60 * 1000表示一天的毫秒数，所以在这里，我们从给定的日期中减去一天。

## 4. 使用Joda-Time

类似地，我们可以使用[Joda-Time API](https://www.baeldung.com/joda-time)来解决我们的核心问题。首先，我们需要将其[依赖](https://mvnrepository.com/artifact/joda-time/joda-time)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10</version>
</dependency>
```

Joda-Time是Java 8发布之前日期和时间操作的事实标准，它提供了自己的LocalDate类版本：

```java
@Test
void givenDate_whenUsingJodaTimeLocalDateClass_thenReturnYesterday() {
    org.joda.time.LocalDate localDate = new org.joda.time.LocalDate(2023, 12, 20);
    org.joda.time.LocalDate yesterdayDate = localDate.minusDays(1);
    org.joda.time.LocalDate expectedYesterdayDate = new org.joda.time.LocalDate(2023, 12, 19);

    assertEquals(expectedYesterdayDate, yesterdayDate);
}
```

简而言之，**该类提供了完全相同的方法minusDays()，我们可以使用它来减去特定的天数，正如它的名称所示**。

## 5. 使用Apache Commons Lang3

另一方面，我们可以选择[Apache Commons Lang3](https://www.baeldung.com/java-commons-lang-3)库。与往常一样，在开始使用它之前，我们需要添加[Maven依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

现在，让我们演示如何使用这个库来获取昨天的日期：

```java
@Test
void givenDate_whenUsingApacheCommonsLangDateUtils_thenReturnYesterday() {
    Date date = new GregorianCalendar(2023, Calendar.MAY, 16, 4, 0).getTime();
    Date yesterdayDate = DateUtils.addDays(date, -1);
    Date expectedYesterdayDate = new GregorianCalendar(2023, Calendar.MAY, 15, 4, 0).getTime();

    assertEquals(expectedYesterdayDate, yesterdayDate);
}
```

Apache Commons Lang3库提供了DateUtils()类用于日期时间操作，**该实用程序类提供了addDays()方法来增加天数，在本例中为-1**。

请注意，该方法返回一个新的Date对象，原始给定的日期保持不变。

## 6. 总结

在这篇短文中，我们详细讲解了如何在Java中获取昨天的日期。我们演示了如何使用核心Java类来实现这一点；然后，我们展示了如何使用外部库，例如Apache Commons Lang3和Joda-Time。