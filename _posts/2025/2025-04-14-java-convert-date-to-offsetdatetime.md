---
layout: post
title:  将Java Date转换为OffsetDateTime
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在本教程中，我们将了解Date和OffsetDateTime之间的区别，我们还将学习如何在两者之间进行转换。

## 2. Date和OffsetDateTime之间的区别

OffsetDateTime是在JDK 8中引入的，作为[java.util.Date](https://www.baeldung.com/java-util-date-sql-date)的现代替代品。

**OffsetDateTime是一个线程安全的类，它将日期和时间存储到纳秒的精度**。另一方面，Date不是线程安全的，它将时间存储到毫秒的精度。

OffsetDateTime是一个基于值的类，这意味着我们在[比较引用](https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/lang/doc-files/ValueBased.html)时需要使用equals而不是典型的==。

OffsetDateTime的toString方法的输出是ISO-8601格式，而Date的toString是自定义的非标准格式。

让我们对这两个类调用toString来查看区别：

```text
Date: Sat Oct 19 17:12:30 2019
OffsetDateTime: 2019-10-19T17:12:30.174Z
```

Date对象无法存储时区及其对应的偏移量，Date对象唯一包含的内容是自1970年1月1日00:00:00 UTC以来的毫秒数，因此如果我们的时间不是UTC，我们应该[将时区存储在辅助类中](https://www.baeldung.com/java-set-date-time-zone)。相反，OffsetDateTime会在内部存储[ZoneOffset](https://www.baeldung.com/java-zone-offset)。

## 3. 将日期转换为OffsetDateTime

将Date转换为OffsetDateTime非常简单，如果我们的Date是UTC格式，我们可以使用一个表达式进行转换：

```java
Date date = new Date();
OffsetDateTime offsetDateTime = date.toInstant()
    .atOffset(ZoneOffset.UTC);
```

如果原始日期不是UTC，我们可以提供偏移量(存储在辅助对象中，因为如前所述，Date类不能存储时区)。

假设我们的原始日期是+3:30(德黑兰时间)：

```java
int hour = 3;
int minute = 30;
OffsetDateTime offsetDateTime = date.toInstant()
    .atOffset(ZoneOffset.ofHoursMinutes(hour, minute));
```

另外，我们可以使用Date类中的getTimeZoneOffset()来计算当地时间和UTC之间的偏移量(以分钟为单位)：

```java
OffsetDateTime offsetDateTime = date.toInstant()
    .atOffset(ZoneOffset.ofTotalSeconds(date.getTimezoneOffset() * -60));
```

这里，我们将本地时间和UTC时间之间的分钟数转换为秒数，ZoneOffset.ofTotalSeconds()方法接收偏移量，即需要添加到UTC的秒数，以获得本地时间。由于getTimezoneOffset()方法返回的是相反的值(需要从本地时间中减去分钟数才能获得UTC时间)，因此我们将其乘以-60以转换为秒数，并反转符号。

Date对象不存储任何时区信息，但是，调用Date对象的getTimezoneOffset()方法可以获取调用Date对象时的系统默认时区。

OffsetDateTime提供了[许多实用的方法](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/OffsetDateTime.html)，可以在之后使用。例如，我们可以简单地使用getDayOfWeek()、getDayOfMonth()和getDayOfYear()。使用isAfter和isBefore方法比较两个OffsetDateTime对象也非常容易。

最重要的是，**完全避免使用已弃用的Date类是一种很好的做法**。

## 4. 总结

在本教程中，我们介绍了从Date转换为OffsetDateTime的方法。