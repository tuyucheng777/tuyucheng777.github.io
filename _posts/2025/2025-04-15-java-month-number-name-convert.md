---
layout: post
title:  在Java中根据月份名称获取月份数字
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在Java中处理月份时，我们经常利用数字格式，因为它有助于跨不同语言和地区进行标准化，因为月份名称差异很大。

在本简短教程中，我们将学习将月份名称转换为其相应数字的不同方法。首先，我们将了解如何使用JDK执行此操作。然后，我们将讨论如何使用第三方[Joda-Time库](https://www.baeldung.com/joda-time)实现相同的结果。

## 2. 使用Java 8+日期时间API

Java 8因其引入的[新日期时间API](https://www.baeldung.com/java-8-date-time-intro)而备受赞誉，该API旨在弥补旧版日期API的不足。那么，让我们看看如何使用这个新API来解答我们的核心问题。

### 2.1 使用Month

最简单的解决方案是使用Month枚举，**顾名思义，它代表了一年中的12个月。除了文本名称外，每个常量都有一个从1(一月)到12(十二月)的int值**。

那么，让我们看看它的实际效果：

```java
@Test
void givenMonthName_whenUsingMonthEnum_thenReturnNumber() {
    String givenMonthName = "October";
    int expectedMonthNumber = 10;

    int monthNumber = Month.valueOf(givenMonthName.toUpperCase())
        .getValue();

    assertEquals(expectedMonthNumber, monthNumber);
}
```

如上所示，我们使用valueOf()方法获取传入月份名称的int值。由于它是一个枚举，因此在调用valueOf()之前，我们使用toUpperCase()方法将输入转换为大写。

### 2.2 使用ChronoField

ChronoField是另一个枚举，我们可以用它来获取给定月份的数字。**它表示一组标准字段，用于访问时间信息，例如日、月、年**。

那么，让我们在实践中看看：

```java
@Test
void givenMonthName_whenUsingChronoFieldEnum_thenReturnNumber() {
    String givenMonthName = "Sep";
    int expectedMonthNumber = 9;

    int monthNumber = DateTimeFormatter.ofPattern("MMM")
        .withLocale(Locale.ENGLISH)
        .parse(givenMonthName)
        .get(ChronoField.MONTH_OF_YEAR);

    assertEquals(expectedMonthNumber, monthNumber);

}
```

**如我们所见，我们使用了MMM格式，这是使用3个字符的月份缩写形式**。然后，我们将MONTH_OF_YEAR常量传递给get()方法，这样，我们就得到了给定月份“Sep”的数字。

## 3. 使用旧版日期API

在Java 8之前，我们通常使用[Date](https://www.baeldung.com/java-year-month-day#java7)或[Calendar](https://www.baeldung.com/java-gregorian-calendar)来操作或使用时间对象。那么，让我们深入研究一下，如何使用这些遗留类将月份名称转换为其对应的数字：

```java
@Test
void givenMonthName_whenUsingDateLegacyAPI_thenReturnNumber() throws ParseException {
    String givenMonthName = "May";
    int expectedMonthNumber = 5;

    Date date = new SimpleDateFormat("MMM", Locale.ENGLISH).parse(givenMonthName);
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(date);

    int monthNumber = calendar.get(Calendar.MONTH) + 1;

    assertEquals(expectedMonthNumber, monthNumber);
}
```

这里，我们使用[SimpleDateFormat](https://www.baeldung.com/java-simple-date-format)和MMM模式将缩写的月份名称解析为Date对象。此外，我们创建一个Calendar实例，并将表示解析出的月份的Date对象设置为日历的时间。

最后，我们使用get()方法从日历中获取月份。**通常，Calendar.MONTH从0开始(一月为0，二月为1，以此类推)，因此我们加1即可获得正确的月份数字**。

## 4. 使用Joda-Time

Joda-Time是另一个可以考虑的方案，可以帮助我们应对挑战。首先，我们将它的[依赖](https://mvnrepository.com/artifact/joda-time/joda-time)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.7</version>
</dependency>
```

**类似地，Joda-Time提供了一种名为getMonthOfYear()的便捷方法，我们可以使用它来获取给定月份的数字**：

```java
@Test
void givenMonthName_whenUsingJodaTime_thenReturnNumber() {
    String givenMonthName = "April";
    int expectedMonthNumber = 4;

    int monthNumber = DateTimeFormat.forPattern("MMM")
        .withLocale(Locale.ENGLISH)
        .parseDateTime(givenMonthName)
        .getMonthOfYear();

    assertEquals(expectedMonthNumber, monthNumber);
}
```

如上所示，逻辑与之前相同，这里我们使用getMonthOfYear()来获取解析后的月份名称的数值。

## 5. 总结

在这篇短文中，我们探讨了将月份名称转换为相应数字的不同方法。

首先，我们了解了如何使用JDK进行转换。然后，我们学习了如何使用Joda-Time库实现相同的结果。