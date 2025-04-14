---
layout: post
title:  在Java中获取不带时间的日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在这个简短的教程中，我们将展示如何在Java中获取不带时间的日期。

我们将展示如何在Java 8之前和之后执行此操作，因为在Java 8中发布新的时间API后情况变得有些不同。

## 2. Java 8之前

在Java 8之前，除非我们使用像Joda-time这样的第三方库，否则没有直接的方法可以获取不带时间的日期。

这是因为Java中的**Date类表示的是特定时刻的时间，以毫秒为单位**。因此，我们无法忽略Date上的时间信息。

在接下来的部分中，我们将展示一些解决此问题的常见解决方法。

### 2.1 使用Calendar

获取不带时间的日期的最常见方法之一是**使用Calendar类将时间设置为0**，这样，我们就能得到一个干净的日期，其时间设置为一天的开始时间。

让我们通过代码来看一下：

```java
public static Date getDateWithoutTimeUsingCalendar() {
    Calendar calendar = Calendar.getInstance();
    calendar.set(Calendar.HOUR_OF_DAY, 0);
    calendar.set(Calendar.MINUTE, 0);
    calendar.set(Calendar.SECOND, 0);
    calendar.set(Calendar.MILLISECOND, 0);

    return calendar.getTime();
}
```

如果我们调用此方法，我们会得到如下日期：

```text
Sat Jun 23 00:00:00 CEST 2018
```

我们可以看到，它返回一个完整的日期，时间设置为0，但**我们不能忽略时间**。

另外，为了确保返回的Date中没有设置时间，我们可以创建以下测试：

```java
@Test
public void whenGettingDateWithoutTimeUsingCalendar_thenReturnDateWithoutTime() {
    Date dateWithoutTime = DateWithoutTime.getDateWithoutTimeUsingCalendar();

    Calendar calendar = Calendar.getInstance();
    calendar.setTime(dateWithoutTime);
    int day = calendar.get(Calendar.DAY_OF_MONTH);

    calendar.setTimeInMillis(dateWithoutTime.getTime() + MILLISECONDS_PER_DAY - 1);
    assertEquals(day, calendar.get(Calendar.DAY_OF_MONTH));

    calendar.setTimeInMillis(dateWithoutTime.getTime() + MILLISECONDS_PER_DAY);
    assertNotEquals(day, calendar.get(Calendar.DAY_OF_MONTH));
}
```

我们可以看到，当我们在日期上加上一天的毫秒数减一时，我们仍然会得到同一天，但是当我们加上一整天时，我们就会得到第二天。

### 2.2 使用格式化器

另一种方法是**将Date格式化为不带时间的String，然后再将该String转换回Date**。由于String的格式化不带时间，因此转换后的Date会将时间设置为0。

让我们看看它的实现：

```java
public static Date getDateWithoutTimeUsingFormat() throws ParseException {
    SimpleDateFormat formatter = new SimpleDateFormat("dd/MM/yyyy");
    return formatter.parse(formatter.format(new Date()));
}
```

此实现返回的内容与上一节中所示的方法相同：

```text
Sat Jun 23 00:00:00 CEST 2018
```

再次，我们可以使用像以前一样的测试来确保返回的日期中没有时间。

## 3. 使用Java 8

随着Java 8中新时间API的发布，现在有了一种更简便的方法来获取不带时间的日期。这个新API带来的新功能之一是，它提供了多个类来处理带时间或不带时间的日期，甚至只处理带时间的日期。

为了本文的目的，我们将重点介绍LocalDate类，它正好代表了我们所需要的，即不带时间的日期。

让我们通过这个例子来看一下：

```java
public static LocalDate getLocalDate() {
    return LocalDate.now();
}
```

此方法返回具有以下日期表示的LocalDate对象：

```text
2018-06-23
```

我们可以看到，它只返回一个日期，时间被完全忽略。

再次，让我们像以前一样测试它，以确保此方法按预期工作：

```java
@Test
public void whenGettingLocalDate_thenReturnDateWithoutTime() {
    LocalDate localDate = DateWithoutTime.getLocalDate();

    long millisLocalDate = localDate
            .atStartOfDay()
            .toInstant(OffsetDateTime
                    .now()
                    .getOffset())
            .toEpochMilli();

    Calendar calendar = Calendar.getInstance();

    calendar.setTimeInMillis(millisLocalDate + MILLISECONDS_PER_DAY - 1);
    assertEquals(
            localDate.getDayOfMonth(),
            calendar.get(Calendar.DAY_OF_MONTH));

    calendar.setTimeInMillis(millisLocalDate + MILLISECONDS_PER_DAY);
    assertNotEquals(
            localDate.getDayOfMonth(),
            calendar.get(Calendar.DAY_OF_MONTH));
}
```

## 4. 总结

在本文中，我们展示了如何在Java中获取不带时间的日期，我们首先展示了如何在Java 8之前版本中执行此操作，以及在Java 8中使用时的操作。

正如我们在文章中实现的示例中所看到的，使用Java 8应该始终是首选，因为它具有处理没有时间的日期的特定表示形式。