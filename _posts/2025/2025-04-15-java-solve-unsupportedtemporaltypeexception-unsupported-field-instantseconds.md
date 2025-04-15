---
layout: post
title:  修复UnsupportedTemporalTypeException：Unsupported Field：InstantSeconds
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

处理日期时，我们经常使用[Date-Time API](https://www.baeldung.com/java-8-date-time-intro)。然而，如果操作或访问时间数据的方式不当，可能会导致错误和异常。其中一个特定的异常是UnsupportedTemporalTypeException：“Unsupported Field:InstantSeconds”，这通常表示指定的时间对象不支持字段[InstantSeconds](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/time/temporal/ChronoField.html#OFFSET_SECONDS)。

因此，在这个简短的教程中，我们将学习如何在使用日期时间API时避免此异常。

## 2. 实际示例

在深入解决方案之前，让我们通过一个实际的例子来了解异常的根本原因。

根据文档，UnsupportedTemporalTypeException表示[ChronoField](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/time/temporal/ChronoField.html)或[ChronoUnit](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/time/temporal/ChronoUnit.html)不受支持。**换句话说，当不支持的字段与不支持该特定字段的时间对象一起使用时，会发生异常**。

通常，堆栈跟踪“Unsupported Field: InstantSeconds”说明了一切，这表明字段InstantSeconds存在问题，该字段表示从[纪元](https://www.baeldung.com/java-localdate-epoch#:~:text=The%20'Epoch'%20in%20Java%20refers,Epoch%20will%20have%20negative%20values.)开始连续计数的秒数的概念。

简而言之，并非所有Date-Time API提供的时间对象都支持此字段。**例如，尝试将涉及InstantSeconds的操作应用于[LocalDateTime](https://www.baeldung.com/java-instant-vs-localdatetime#java-localdatetime)、[LocalDate](https://www.baeldung.com/java-creating-localdate-with-values)和[LocalTime](https://www.baeldung.com/java-8-date-time-intro#2-working-with-localtime)会导致UnsupportedTemporalTypeException**。

现在，让我们看看如何在实践中重现该异常。为此，我们尝试将LocalDateTime转换为[Instant](https://www.baeldung.com/java-instant-vs-localdatetime#java-instant)：

```java
@Test
void givenLocalDateTime_whenConvertingToInstant_thenThrowException() {
    assertThatThrownBy(() -> {
        LocalDateTime localDateTime = LocalDateTime.now();
        long seconds = localDateTime.getLong(ChronoField.INSTANT_SECONDS);
        Instant instant = Instant.ofEpochSecond(seconds);
    }).isInstanceOf(UnsupportedTemporalTypeException.class)
        .hasMessage("Unsupported field: InstantSeconds");
}
```

我们可以看到，测试失败了，因为我们尝试使用LocalDateTime的实例访问ChronoField.INSTANT_SECONDS。

简而言之，**LocalDateTime不支持表示单个特定时刻的INSTANT_SECONDS参数，因为同一个LocalDateTime对象可能根据时区的不同而表示多个不同的时刻**。例如，纽约的“2024-06-22T11:20:33”与东京的“2024-06-22T11:20:33”是不同的。

## 3. 解决方案

正如我们之前提到的，LocalDateTime不支持ChronoField.INSTANT_SECONDS的主要原因是它缺乏足够的时区信息来确定全局时间轴上的精确瞬时点。

**因此，最简单的解决方案是在将LocalDateTime转换为Instant之前设置时区。为此，我们可以使用[ZonedDateTime](https://www.baeldung.com/java-format-zoned-datetime-string#date_to_string)类**：

```java
@Test
void givenLocalDateTime_whenConvertingUsingTimeZone_thenDoNotThrowException() {
    LocalDateTime localDateTime = LocalDateTime.now();
    ZonedDateTime zonedDateTime = localDateTime.atZone(ZoneId.systemDefault());

    assertThatCode(() -> {
        Instant instant = zonedDateTime.toInstant();
    }).doesNotThrowAnyException();
}
```

**在这里，我们使用atZone()方法返回由给定LocalDateTime在指定时区形成的ZonedDateTime对象**。然后，我们调用toInstant()根据提供的时区计算日期时间所代表的时刻。

## 4. 总结

在这篇短文中，我们了解了UnsupportedTemporalTypeException的根本原因。此外，我们还了解了如何在实践中重现和修复它。