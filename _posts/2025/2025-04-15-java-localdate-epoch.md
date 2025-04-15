---
layout: post
title:  Java LocalDate和Epoch之间的转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将演示Java的LocalDate和Epoch之间的相互转换。

## 2. Epoch与LocalDate

要进行转换，理解Epoch和LocalDate背后的概念至关重要。Java中的“Epoch”指的是1970-01-01T00:00:00Z这一时刻，Epoch之后的时刻将为正值。同样，Epoch之前的任何时刻都将为负值。

**Epoch、LocalDate和LocalDateTime的所有实例都与时区相关**，因此，在从一个实例转换为另一个实例时，我们必须知道时区。在Java中，时区可以通过ZoneId类表示。[ZoneId](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/ZoneId.html)可以是通过方法ZoneId.systemDefault()获取的系统时区。或者，也可以通过传递一个已知时区(例如Europe/Amsterdam)的字符串来计算ZoneId。 

## 3. 纪元转为日期/时间

我们可以**根据自纪元以来的毫秒数来计算LocalDate或LocalDateTime**，或者，计数也可以以秒为单位，或者以纳秒为单位的秒数，此计数的Java数据类型为Long。最后，我们还需要知道时区，让我们看看如何进行转换：

```java
long milliSecondsSinceEpoch = 2131242L;
ZoneId zoneId = ZoneId.of("Europe/Amsterdam");
LocalDate date = Instant.ofEpochMilli(milliSecondsSinceEpoch).atZone(zoneId).toLocalDate();
```

在上面的代码片段中，我们获得了自阿姆斯特丹时区纪元以来的毫秒数，因此我们可以使用Instant类的ofEpochMilli()方法来获取LocalDate值。否则，如果我们想要获取时间而不是日期，那么我们可以这样写：

```java
LocalDateTime time = Instant.ofEpochMilli(milliSecondsSinceEpoch).atZone(zoneId).toLocalDateTime();
```

在上面的代码片段中，我们使用了相同的方法，但使用了toLocalDateTime方法。

## 4. 日期/时间转为纪元

如果我们在给定时区中有一个LocalDate日期，那么我们就可以获取以秒为单位的纪元时间，让我们看看如何操作：

```java
ZoneId zoneId = ZoneId.of("Europe/Tallinn"); 
LocalDate date = LocalDate.now(); 
long EpochMilliSecondsAtDate = date.atStartOfDay(zoneId).toInstant().toEpochMilli();
```

在上面的例子中，我们获取了当前日期的纪元秒数以及系统当前所在的时区。请注意，**我们只能获取当天开始的纪元计数**，这是因为LocalDate没有任何时间值信息。或者，如果我们有时间部分，我们可以获取给定时刻的精确纪元计数：

```java
LocalDateTime localDateTime = LocalDateTime.parse("2019-11-15T13:15:30");
long epochMilliSecondsAtTime = localDateTime.atZone(zoneId).toInstant().toEpochMilli();
```

## 5. 总结

在本文中，我们探讨了如何将Epoch转换为LocalDate和LocalDateTime，我们还演示了如何将LocalDate或LocalDateTime转换为Epoch。