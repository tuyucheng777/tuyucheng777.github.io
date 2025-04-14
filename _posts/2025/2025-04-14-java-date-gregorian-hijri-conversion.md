---
layout: post
title:  在Java中将公历日期转换为回历日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

公历和回历代表两种不同的时间测量系统。

在本教程中，我们将研究将公历日期转换为回历日期的各种方法。

## 2. 公历与回历

让我们了解一下公历和回历的区别，[公历](https://www.baeldung.com/java-gregorian-calendar#gregorian-calendar)遵循太阳年，包含12个月，每个月的长度固定。回历遵循阴历年，包含12个月，每个月的长度在29天和30天之间交替。

在回历中，每个月的长度取决于月球绕地球公转一周的时间。**公历有365或366天，而回历有354或355天，这意味着回历年比公历年大约短11天**。

## 3. 使用HijrahDate类

在这个方法中，我们将使用java.time.chrono包中的HijrahDate类。该类是在Java 8中引入的，用于现代日期和时间操作，它提供了多种创建和操作回历日期的方法。

### 3.1 使用from()方法

我们将使用HijrahDate类的from()方法将日期从公历转换为回历，此方法以表示公历日期的[LocalDate](https://www.baeldung.com/java-creating-localdate-with-values)对象作为输入，并返回一个HijriDate对象：

```java
public HijrahDate usingFromMethod(LocalDate gregorianDate) {
    return HijrahDate.from(gregorianDate);
}
```

现在，让我们运行测试：

```java
void givenGregorianDate_whenUsingFromMethod_thenConvertHijriDate() {
    LocalDate gregorianDate = LocalDate.of(2013, 3, 31);
    HijrahDate hijriDate = GregorianToHijriDateConverter.usingFromMethod(gregorianDate);
    assertEquals(1434, hijriDate.get(ChronoField.YEAR));
    assertEquals(5, hijriDate.get(ChronoField.MONTH_OF_YEAR));
    assertEquals(19, hijriDate.get(ChronoField.DAY_OF_MONTH));
}
```

### 3.2 使用HijrahChronology类

在这种方法中，我们将使用java.time.chrono.HijrahChronology类，它代表Hijri(回历)日历系统。

HijrahChoronology.INSTANCE方法创建回历系统的实例，我们将使用它来创建ChronoLocalDate对象，以便将公历日期转换为回历日期：

```java
public HijrahDate usingHijrahChronology(LocalDate gregorianDate) {
    HijrahChronology hijrahChronology = HijrahChronology.INSTANCE;
    ChronoLocalDate hijriChronoLocalDate = hijrahChronology.date(gregorianDate);
    return HijrahDate.from(hijriChronoLocalDate);
}
```

现在，让我们测试一下这种方法：

```java
void givenGregorianDate_whenUsingHijrahChronologyClass_thenConvertHijriDate() {
    LocalDate gregorianDate = LocalDate.of(2013, 3, 31);
    HijrahDate hijriDate = GregorianToHijriDateConverter.usingHijrahChronology(gregorianDate);
    assertEquals(1434, hijriDate.get(ChronoField.YEAR));
    assertEquals(5, hijriDate.get(ChronoField.MONTH_OF_YEAR));
    assertEquals(19, hijriDate.get(ChronoField.DAY_OF_MONTH));
}
```

## 4. 使用Joda-Time

[Joda-Time](https://www.baeldung.com/joda-time)是Java中流行的日期和时间操作库，它为标准Java日期和时间API提供了更直观的替代方案。

**在Joda-Time中，IslamicChronology类代表伊斯兰回历，我们将使用DateTime的withChronology()方法和一个IslamicChronology实例将公历日期转换为伊斯兰回历日期**：

```java
public DateTime usingJodaDate(DateTime gregorianDate) {
    return gregorianDate.withChronology(IslamicChronology.getInstance());
}
```

现在，让我们测试一下这种方法：

```java
void givenGregorianDate_whenUsingJodaDate_thenConvertHijriDate() {
    DateTime gregorianDate = new DateTime(2013, 3, 31, 0, 0, 0);
    DateTime hijriDate = GregorianToHijriDateConverter.usingJodaDate(gregorianDate);
    assertEquals(1434, hijriDate.getYear());
    assertEquals(5, hijriDate.getMonthOfYear());
    assertEquals(19, hijriDate.getDayOfMonth());
}
```

## 5. 使用UmmalquraCalendar类

[ummalqura-calendar](https://github.com/msarhan/ummalqura-calendar)库具有UmmalquraCalendar类，该类派生自[Java 8](https://www.baeldung.com/java-8-date-time-intro)。要包含ummalqura-calendar库，我们需要添加以下[依赖](https://mvnrepository.com/artifact/com.github.msarhan/ummalqura-calendar)：

```xml
<dependency>
    <groupId>com.github.msarhan</groupId>
    <artifactId>ummalqura-calendar</artifactId>
    <version>2.0.2</version>
</dependency>
```

我们将使用它的setTime()方法执行公历到回历日期的转换：

```java
public UmmalquraCalendar usingUmmalquraCalendar(GregorianCalendar gregorianCalendar) throws ParseException {
    UmmalquraCalendar hijriCalendar = new UmmalquraCalendar();
    hijriCalendar.setTime(gregorianCalendar.getTime());
    return hijriCalendar;
}
```

现在，让我们测试一下这种方法：

```java
void givenGregorianDate_whenUsingUmmalquraCalendar_thenConvertHijriDate() throws ParseException {
    GregorianCalendar gregorianCalenar = new GregorianCalendar(2013, Calendar.MARCH, 31);
    UmmalquraCalendar ummalquraCalendar = GregorianToHijriDateConverter.usingUmmalquraCalendar(gregorianCalenar);
    assertEquals(1434, ummalquraCalendar.get(Calendar.YEAR));
    assertEquals(5, ummalquraCalendar.get(Calendar.MONTH) + 1);
    assertEquals(19, ummalquraCalendar.get(Calendar.DAY_OF_MONTH));
}
```

## 6. 总结

在本教程中，我们讨论了将公历日期转换为回历日期的各种方法。