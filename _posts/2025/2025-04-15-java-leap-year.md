---
layout: post
title:  在Java中查找闰年
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

 在本教程中，我们将演示几种在Java中确定给定年份是否为[闰年](https://en.wikipedia.org/wiki/Leap_year)的方法。

**闰年是指能被4和400整除而没有余数的年份**，因此，能被100整除但不能被400整除的年份不符合闰年条件，即使它们能被4整除。

## 2. 使用Java 8之前的Calendar API

从Java 1.1开始，[GregorianCalendar](https://www.baeldung.com/java-gregorian-calendar)类允许我们检查某一年份是否是闰年：

```java
public boolean isLeapYear(int year);
```

正如我们所料，如果给定的年份是闰年，则此方法返回true，如果给定的年份是非闰年，则返回false。

**公元前(BC)年份需要以负值形式传递**，计算公式为1 - 年。例如，公元前3年表示为-2，因为1 - 3 = -2。

## 3. 使用Java 8+日期/时间API

Java 8引入了java.time包，它具有更好的[日期和时间API](https://www.baeldung.com/java-8-date-time-intro)。

java.time包中的[Year](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/Year.html)类有一个静态方法来检查给定的年份是否是闰年：

```java
public static boolean isLeap(long year);
```

它也有一个实例方法来做同样的事情：

```java
public boolean isLeap();
```

## 4. 使用Joda-Time API

[Joda-Time API](https://www.baeldung.com/joda-time)是Java项目中用于日期和时间实用程序的最常用第三方库之一，自Java 8以来，该库处于可维护状态，如[Joda-Time GitHub源代码库](https://github.com/JodaOrg/joda-time#joda-time)中所述。

Joda-Time没有预定义的API方法来查找闰年，但是，我们可以使用[LocalDate](https://www.joda.org/joda-time/apidocs/org/joda/time/LocalDate.html)和[Days](https://www.joda.org/joda-time/apidocs/org/joda/time/Days.html)类来检查闰年：

```java
LocalDate localDate = new LocalDate(2020, 1, 31);
int numberOfDays = Days.daysBetween(localDate, localDate.plusYears(1)).getDays();

boolean isLeapYear = (numberOfDays > 365) ? true : false;
```

## 5. 总结

在本教程中，我们了解了什么是闰年、查找闰年的逻辑以及可以用来检查闰年的几个Java API。