---
layout: post
title:  检查两个Java日期是否为同一天
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本快速教程中，**我们将学习几种不同的方法来检查两个java.util.Date对象是否具有相同的日期**。

我们将首先考虑使用核心Java(即Java 8功能)的解决方案，然后再考虑几个Java 8之前的替代方案。

最后，我们还将研究一些外部库-**Apache Commons Lang、Joda-Time和Date4J**。

## 2. 核心Java

**Date类表示特定的时间点，精度为毫秒**。要确定两个Date对象是否包含同一天，**我们需要检查两个对象的“年-月-日”是否相同，并忽略时间因素**。

### 2.1 使用LocalDate

借助[Java 8中新的Date-Time API](https://www.baeldung.com/java-8-date-time-intro)，我们可以使用LocalDate对象。这是一个不可变的对象，表示没有时间的日期。

让我们看看如何使用此类检查两个Date对象是否是同一天：

```java
public static boolean isSameDay(Date date1, Date date2) {
    LocalDate localDate1 = date1.toInstant()
            .atZone(ZoneId.systemDefault())
            .toLocalDate();
    LocalDate localDate2 = date2.toInstant()
            .atZone(ZoneId.systemDefault())
            .toLocalDate();
    return localDate1.isEqual(localDate2);
}
```

在此示例中，**我们使用默认时区将两个Date对象转换为LocalDate**。转换后，我们只需**使用isEqual方法检查LocalDate对象是否相等**。

因此，使用这种方法，我们将能够确定两个Date对象是否包含同一天。

### 2.2 使用Instant

在上面的示例中，我们在将Date对象转换为LocalDate对象时使用了Instant作为中间对象，**Java 8中引入的Instant表示特定的时间点**。

我们可以直接**将Instant对象截断为DAYS单位**，将时间字段值设置为0，然后我们可以比较它们：

```java
public static boolean isSameDayUsingInstant(Date date1, Date date2) {
    Instant instant1 = date1.toInstant()
            .truncatedTo(ChronoUnit.DAYS);
    Instant instant2 = date2.toInstant()
            .truncatedTo(ChronoUnit.DAYS);
    return instant1.equals(instant2);
}
```

### 2.3 使用SimpleDateFormat

从Java早期版本开始，我们就可以使用[SimpleDateFormat](https://www.baeldung.com/java-simple-date-format)类在Date和String对象表示之间进行转换。该类支持多种转换模式，**在本例中，我们将使用“yyyyMMdd”这种模式**。

使用这个，我们将格式化日期，将其转换为字符串对象，然后使用标准equals方法对它们进行比较：

```java
public static boolean isSameDay(Date date1, Date date2) {
    SimpleDateFormat fmt = new SimpleDateFormat("yyyyMMdd");
    return fmt.format(date1).equals(fmt.format(date2));
}
```

### 2.4 使用Calendar

Calendar类提供了获取特定时刻不同日期时间单位的值的方法。

首先，我们需要创建一个Calendar实例，并使用每个提供的日期设置Calendar对象的时间。然后，我们可以分别查询并比较Year-Month-Day属性，以确定Date对象是否为同一天：

```java
public static boolean isSameDay(Date date1, Date date2) {
    Calendar calendar1 = Calendar.getInstance();
    calendar1.setTime(date1);
    Calendar calendar2 = Calendar.getInstance();
    calendar2.setTime(date2);
    return calendar1.get(Calendar.YEAR) == calendar2.get(Calendar.YEAR)
        && calendar1.get(Calendar.MONTH) == calendar2.get(Calendar.MONTH)
        && calendar1.get(Calendar.DAY_OF_MONTH) == calendar2.get(Calendar.DAY_OF_MONTH);
}
```

## 3. 外部库

现在我们已经很好地了解了如何使用核心Java提供的新旧API来比较Date对象，让我们看一些外部库。

### 3.1 Apache Commons Lang DateUtils

**DateUtils类提供了许多有用的实用程序，使使用旧式日历和日期对象变得更加容易**。

[Apache Commons Lang](https://commons.apache.org/proper/commons-lang/)工件可从[Maven Central](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)获得：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

然后我们可以简单地使用DateUtils中的方法isSameDay：

```java
DateUtils.isSameDay(date1, date2);
```

### 3.2 Joda-Time库

核心Java日期和时间库的替代方案是[Joda-Time](https://www.baeldung.com/joda-time)，**这个广泛使用的[库](http://www.joda.org/joda-time/)在使用日期和时间时是一个很好的替代方案**。

该工件可以在[Maven Central](https://mvnrepository.com/artifact/joda-time/joda-time)上找到：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

在这个库中，**org.joda.time.LocalDate表示不带时间的日期**，因此，我们可以从java.util.Date对象构造LocalDate对象，然后比较它们：

```java
public static boolean isSameDay(Date date1, Date date2) {
    org.joda.time.LocalDate localDate1 = new org.joda.time.LocalDate(date1);
    org.joda.time.LocalDate localDate2 = new org.joda.time.LocalDate(date2);
    return localDate1.equals(localDate2);
}
```

### 3.3 Date4J库

[Date4j](https://github.com/johanley/date4j)还提供了一个我们可以使用的直接、简单的实现。

同样，它也可以从[Maven Central](https://mvnrepository.com/artifact/com.darwinsys/hirondelle-date4j)获得：

```xml
<dependency>
    <groupId>com.darwinsys</groupId>
    <artifactId>hirondelle-date4j</artifactId>
    <version>1.5.1</version>
</dependency>
```

使用这个库，**我们需要从java.util.Date对象构造DateTime对象，然后可以简单地使用isSameDayAs方法**：

```java
public static boolean isSameDay(Date date1, Date date2) {
    DateTime dateObject1 = DateTime.forInstant(date1.getTime(), TimeZone.getDefault());
    DateTime dateObject2 = DateTime.forInstant(date2.getTime(), TimeZone.getDefault());
    return dateObject1.isSameDayAs(dateObject2);
}
```

## 4. 总结

在本快速教程中，我们探讨了几种检查两个java.util.Date对象是否是同一天的方法。