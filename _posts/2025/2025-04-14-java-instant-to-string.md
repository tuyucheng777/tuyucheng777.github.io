---
layout: post
title:  在Java中将Instant数据格式化为String
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将学习如何**在Java中将顺时格式化为字符串**。

首先，我们先来了解一下Java中Instant的概念，然后，我们将演示如何使用Java核心库和第三方库(例如Joda-Time)来解答我们的核心问题。

## 2. 使用核心Java格式化Instant

根据[Java文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/Instant.html)，瞬时是从Java纪元1970-01-01T00:00:00Z测量的时间戳。

Java 8包含一个名为Instant的便捷类，用于表示时间轴上的特定瞬时点。通常，我们可以在应用程序中使用此类来记录事件时间戳。

现在我们知道了Java中的Instant是什么，让我们看看如何将其转换为String对象。

### 2.1 使用DateTimeFormatter类

一般来说，我们需要一个格式化程序来格式化Instant对象。幸运的是，Java 8引入了[DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter)类来统一格式化日期和时间。

基本上，DateTimeFormatter提供了format()方法来完成这个任务。

简而言之，**DateTimeFormatter需要时区来格式化时刻**，如果没有时区，它将无法将时刻转换为人类可读的日期/时间字段。

例如，假设我们要使用dd.MM.yyyy格式显示我们的Instant实例：

```java
public class FormatInstantUnitTest {

    private static final String PATTERN_FORMAT = "dd.MM.yyyy";

    @Test
    public void givenInstant_whenUsingDateTimeFormatter_thenFormat() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(PATTERN_FORMAT)
                .withZone(ZoneId.systemDefault());

        Instant instant = Instant.parse("2022-02-15T18:35:24.00Z");
        String formattedInstant = formatter.format(instant);

        assertThat(formattedInstant).isEqualTo("15.02.2022");
    }
    // ...
}
```

如上所示，我们可以使用withZone()方法来指定时区。

请记住，**未能指定时区将导致[UnsupportedTemporalTypeException](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/temporal/UnsupportedTemporalTypeException.html)**：

```java
@Test(expected = UnsupportedTemporalTypeException.class)
public void givenInstant_whenNotSpecifyingTimeZone_thenThrowException() {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern(PATTERN_FORMAT);

    Instant instant = Instant.now();
    formatter.format(instant);
}
```

### 2.2 使用toString()方法

另一个解决方案是使用toString()方法来获取Instant对象的字符串表示形式。

让我们使用测试用例来举例说明toString()方法的使用：

```java
@Test
public void givenInstant_whenUsingToString_thenFormat() {
    Instant instant = Instant.ofEpochMilli(1641828224000L);
    String formattedInstant = instant.toString();

    assertThat(formattedInstant).isEqualTo("2022-01-10T15:23:44Z");
}
```

**这种方法的局限性在于我们不能使用自定义的、人性化的格式来显示即时信息**。

## 3. Joda-Time库

或者，我们可以使用[Joda-Time API](https://www.baeldung.com/joda-time)来实现相同的目标，该库提供了一组现成的类和接口，用于在Java中操作日期和时间。

在这些类中，我们可以使用[DateTimeFormat](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormatter.html)类。顾名思义，**该类可用于将日期/时间数据格式化或解析为字符串**。

让我们说明如何使用DateTimeFormatter将时刻转换为字符串：

```java
@Test
public void givenInstant_whenUsingJodaTime_thenFormat() {
    org.joda.time.Instant instant = new org.joda.time.Instant("2022-03-20T10:11:12");

    String formattedInstant = DateTimeFormat.forPattern(PATTERN_FORMAT)
            .print(instant);

    assertThat(formattedInstant).isEqualTo("20.03.2022");
}
```

我们可以看到，DateTimeFormatter提供了forPattern()来指定格式化模式，以及print()来格式化Instant对象。

## 4. 总结

在本文中，我们深入介绍了如何在Java中将瞬时格式化为字符串。

我们探索了几种使用核心Java方法实现此目标的方法；然后，我们解释了如何使用Joda-Time库实现同样的目标。