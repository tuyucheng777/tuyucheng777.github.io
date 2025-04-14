---
layout: post
title:  在Java中将小时数添加到日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在Java 8之前，java.util.Date是Java中表示日期时间值最常用的类之一。

然后，Java 8引入了java.time.LocalDateTime和java.time.ZonedDateTime。Java 8还允许我们使用java.time.Instant在时间轴上表示特定时间。

在本教程中，我们将学习**在Java中对给定的日期时间进行加或减n小时的操作**。首先，我们将介绍一些标准的Java日期时间相关类，然后再展示一些第三方选项。

要了解有关Java 8 DateTime API的更多信息，我们建议阅读[这篇文章](https://www.baeldung.com/java-8-date-time-intro)。

## 2. java.util.Date

如果我们使用的是Java 7或更低版本，我们可以使用[java.util.Date](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Date.html)和[java.util.Calendar](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Calendar.html)类进行大多数与日期时间相关的处理。

让我们看看如何向给定的Date对象添加n小时：

```java
public Date addHoursToJavaUtilDate(Date date, int hours) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(date);
    calendar.add(Calendar.HOUR_OF_DAY, hours);
    return calendar.getTime();
}
```

请注意，**Calendar.HOUR_OF_DAY指的是24小时制**。

上述方法返回一个新的Date对象，其值将是(date + hours)或(date - hours)，具体取决于我们传递的是正数小时值还是负数小时值。

**假设我们有一个Java 8应用程序，但我们仍然希望使用java.util.Date实例来工作**。

对于这种情况，我们可以选择采取以下替代方法：

1. 使用java.util.Date toInstant()方法将Date对象转换为java.time.Instant实例
2. 使用plus()方法向java.time.Instant对象添加特定的Duration
3. 通过将java.time.Instant对象传递给java.util.Date.from()方法来恢复我们的java.util.Date实例

让我们快速看一下这种方法：

```java
@Test
public void givenJavaUtilDate_whenUsingToInstant_thenAddHours() {
    Date actualDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 5, 0)
        .getTime();
    Date expectedDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 7, 0)
        .getTime();

    assertThat(Date.from(actualDate.toInstant().plus(Duration.ofHours(2))))
        .isEqualTo(expectedDate);
}
```

但是请注意，**始终建议对Java 8或更高版本上的所有应用程序使用新的DateTime API**。

## 3. java.time.LocalDateTime/ZonedDateTime

在Java 8或更高版本中，向[java.time.LocalDateTime](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/LocalDateTime.html)或[java.time.ZonedDateTime](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/ZonedDateTime.html)实例添加小时数非常简单，只需使用plusHours()方法即可：

```java
@Test
public void givenLocalDateTime_whenUsingPlusHours_thenAddHours() {
    LocalDateTime actualDateTime = LocalDateTime
        .of(2018, Month.JUNE, 25, 5, 0);
    LocalDateTime expectedDateTime = LocalDateTime
        .of(2018, Month.JUNE, 25, 10, 0);

    assertThat(actualDateTime.plusHours(5)).isEqualTo(expectedDateTime);
}
```

如果我们想减少几个小时怎么办？

将小时数的负值传递给plusHours()方法就可以了，不过，建议使用minusHours()方法：

```java
@Test
public void givenLocalDateTime_whenUsingMinusHours_thenSubtractHours() {
    LocalDateTime actualDateTime = LocalDateTime
        .of(2018, Month.JUNE, 25, 5, 0);
    LocalDateTime expectedDateTime = LocalDateTime
        .of(2018, Month.JUNE, 25, 3, 0);
   
    assertThat(actualDateTime.minusHours(2)).isEqualTo(expectedDateTime);
}
```

java.time.ZonedDateTime中的plusHours()和minusHours()方法的工作方式完全相同。

## 4. java.time.Instant

我们知道，Java 8 DateTime API中引入的[java.time.Instant](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/Instant.html)代表时间轴上的特定时刻。

**要向Instant对象添加一些小时，我们可以将其plus()方法与[java.time.temporal.TemporalAmount](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/temporal/TemporalAmount.html)一起使用**：

```java
@Test
public void givenInstant_whenUsingAddHoursToInstant_thenAddHours() {
    Instant actualValue = Instant.parse("2018-06-25T05:12:35Z");
    Instant expectedValue = Instant.parse("2018-06-25T07:12:35Z");

    assertThat(actualValue.plus(2, ChronoUnit.HOURS))
        .isEqualTo(expectedValue);
}
```

类似地，minus()方法可用于减去特定的TemporalAmount。

## 5. Apache Commons DateUtils

Apache Commons Lang库中的[DateUtils](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/time/DateUtils.html#addHours-java.util.Date-int-)类公开了一个静态addHours()方法：

```java
public static Date addHours(Date date, int amount)
```

该方法接收一个java.util.Date对象以及我们希望添加到其中的数值，该值可以是正数也可以是负数。

返回一个新的java.util.Date对象作为结果：

```java
@Test
public void givenJavaUtilDate_whenUsingApacheCommons_thenAddHours() {
    Date actualDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 5, 0)
        .getTime();
    Date expectedDate = new GregorianCalendar(2018, Calendar.JUNE, 25, 7, 0)
        .getTime();

    assertThat(DateUtils.addHours(actualDate, 2)).isEqualTo(expectedDate);
}
```

Apache Commons Lang的最新版本可在[Maven Central](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)上找到。

## 6. Joda Time

[Joda Time](http://www.joda.org/joda-time/)是Java 8 DateTime API的替代品，并提供自己的DateTime实现。

大多数与DateTime相关的类都公开了plusHours()和minusHours()方法来帮助我们从DateTime对象中添加或减去给定的小时数。

让我们看一个例子：

```java
@Test
public void givenJodaDateTime_whenUsingPlusHoursToDateTime_thenAddHours() {
    DateTime actualDateTime = new DateTime(2018, 5, 25, 5, 0);
    DateTime expectedDateTime = new DateTime(2018, 5, 25, 7, 0);

    assertThat(actualDateTime.plusHours(2)).isEqualTo(expectedDateTime);
}
```

可以在[Maven Central](https://mvnrepository.com/artifact/joda-time/joda-time)查看Joda Time的最新可用版本。

## 7. 总结

在本教程中，我们介绍了几种在标准Java日期时间值中添加或减去给定小时数的方法。