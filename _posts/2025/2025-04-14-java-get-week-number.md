---
layout: post
title:  获取任意日期的周数
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在本文中，我们将学习Java中用于获取给定日期的周数的几个选项。首先，我们将介绍一些使用Java 8之前类的遗留代码的选项。之后，我们将介绍Java 8中引入的java.time包中较新的[日期时间API](https://www.baeldung.com/java-8-date-time-intro)。

## 2. Java 8之前

在Java 8之前，日期和时间计算主要使用Date和Calendar类。通常我们创建一个Calendar类，然后可以使用不同的常量从中提取所需的信息。

### 2.1 使用Calendar字段获取周数

让我们看一下第一个例子：

```java
Calendar calendar = Calendar.getInstance(locale); 
calendar.set(year, month, day); 
int weekOfYear = calendar.get(Calendar.WEEK_OF_YEAR);
```

我们只需根据给定的Locale创建一个Calendar实例，并设置年、月、日，最后从日历对象中获取WEEK_OF_YEAR字段，这将返回当前年份的周数。

现在，让我们看一下如何从我们的一个单元测试中调用此方法：

```java
@Test
public void givenDateUsingFieldsAndLocaleItaly_whenGetWeekNumber_thenWeekIsReturnedCorrectly() {
    Calendar calendar = Calendar.getInstance(Locale.ITALY);
    calendar.set(2020, 10, 22);

    assertEquals(47, calendar.get(Calendar.WEEK_OF_YEAR));
}
```

采用这种方法时需要小心，因为Calendar类中的月份字段是从0开始的，**这意味着如果我们想指定十二月，就需要使用数字11，这常常会导致混淆**。

### 2.2 使用Locale设置获取周数

在倒数第二个示例中，我们将看看将一些附加设置应用到Calendar会产生什么效果：

```java
Calendar calendar = Calendar.getInstance();
calendar.setFirstDayOfWeek(firstDayOfWeek);
calendar.setMinimalDaysInFirstWeek(minimalDaysInFirstWeek);
calendar.set(year, month, day);

int weekOfYear = calendar.get(Calendar.WEEK_OF_YEAR);
```

Calendar类定义了两种方法：

- setFirstDayOfWeek
- setMinimalDaysInFirstWeek

这些方法会影响我们如何计算周数，**通常，这两个值都是在创建Calendar时从Locale中获取的**，但也可以手动设置一周的第一天和一年中第一周的最少天数。

### 2.3 区域差异

区域设置在如何计算周数方面起着重要作用：

```java
@Test
public void givenDateUsingFieldsAndLocaleCanada_whenGetWeekNumber_thenWeekIsReturnedCorrectly() {
    Calendar calendar = Calendar.getInstance(Locale.CANADA);
    calendar.set(2020, 10, 22);

    assertEquals(48, calendar.get(Calendar.WEEK_OF_YEAR));
}
```

在这个单元测试中，我们只将Calendar的语言环境更改为使用Locale.CANADA而不是Locale.ITALY，现在返回的周数是48而不是47。

两个结果都是正确的，**如前所述，发生这种情况是因为每个Locale对setFirstDayOfWeek和setMinimalDaysInFirstWeek方法的设置不同**。

## 3. Java 8日期时间API

Java 8引入了新的[日期和时间API](https://www.baeldung.com/java-8-date-time-intro)，以解决旧版java.util.Date和 java.util.Calendar的缺点。

在本节中，我们将了解使用此较新的API从日期中获取周数的一些选项。

### 3.1 使用数值获取周数

同样，正如我们之前在Calendar中看到的，我们也可以将年、月、日的值直接传递给LocalDate：

```java
LocalDate date = LocalDate.of(year, month, day);
int weekOfYear = date.get(WeekFields.of(locale).weekOfYear());
```

**与Java 8之前的示例相比，其好处在于我们不存在月份字段从0开始的问题**。

### 3.2 使用Chronofield获取周数

在最后一个例子中，我们将看到如何**使用实现TemporalField接口的ChronoField枚举**：

```java
LocalDate date = LocalDate.of(year, month, day);
int weekOfYear = date.get(ChronoField.ALIGNED_WEEK_OF_YEAR);
```

此示例类似于使用我们之前看到的Calendar.WEEK_OF_YEAR int常量，但使用的是ChronoField.ALIGNED_WEEK_OF_YEAR。

## 4. 总结

在本快速教程中，我们说明了使用纯Java从日期获取周数的几种方法。