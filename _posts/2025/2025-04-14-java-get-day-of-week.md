---
layout: post
title:  如何在Java中通过传递具体日期来确定星期几
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在这个简短的教程中，我们将了解如何从Java日期中提取星期几作为数字和文本。

## 2. 问题

业务逻辑通常需要星期几，为什么？首先，工作日和周末的工作时间和服务水平有所不同。因此，对于很多系统来说，获取数字形式的日期是必要的，但我们也可能需要将日期以文本形式显示。

那么，我们如何从Java中的日期中提取星期几？

## 3. 使用java.util.Date的解决方案

java.util.Date自Java 1.0以来一直是Java日期类，从Java 7或更低版本开始的代码可能使用此类。

### 3.1 数字形式的星期几

首先，我们**使用java.util.Calendar将日期提取为数字**：

```java
public static int getDayNumberOld(Date date) {
    Calendar cal = Calendar.getInstance();
    cal.setTime(date);
    return cal.get(Calendar.DAY_OF_WEEK);
}
```

结果数字的**范围是1(星期日)到7(星期六)**，Calendar为此定义了常量：Calendar.SUNDAY-Calendar.SATURDAY。

### 3.2 文本形式的星期几

现在我们将日期提取为文本，我们传入Locale来确定语言：

```java
public static String getDayStringOld(Date date, Locale locale) {
    DateFormat formatter = new SimpleDateFormat("EEEE", locale);
    return formatter.format(date);
}
```

这将返回你语言的全天信息，例如英语中的“Monday”或德语中的“Montag”。

## 4. 使用java.time.LocalDate的解决方案

[Java 8](https://www.baeldung.com/java-8-date-time-intro)彻底改进了日期和时间处理，并引入了java.time.LocalDate类。因此，**仅在Java 8或更高版本上运行的Java项目应该使用此类**。

### 4.1 数字形式的星期几

现在将日期提取为数字很简单：

```java
public static int getDayNumberNew(LocalDate date) {
    DayOfWeek day = date.getDayOfWeek();
    return day.getValue();
}
```

结果数字仍然在1到7之间，但这次，**星期一是1，星期日是7**。星期几有自己的枚举DayOfWeek，正如预期的那样，枚举值为MONDAY– SUNDAY。

### 4.2 文本形式的星期几

现在我们再次将日期提取为文本，我们也传入一个Locale值：

```java
public static String getDayStringNew(LocalDate date, Locale locale) {
    DayOfWeek day = date.getDayOfWeek();
    return day.getDisplayName(TextStyle.FULL, locale);
}
```

与java.util.Date一样，这将以所选语言返回完整的日期。

## 5. 总结

在本文中，我们从Java日期中提取了星期几，我们还了解了如何使用java.util.Date和 java.time.LocalDate返回数字和文本。