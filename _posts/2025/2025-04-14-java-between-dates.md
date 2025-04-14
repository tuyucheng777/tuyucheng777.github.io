---
layout: post
title:  如何获取两个日期之间的所有日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

Java 8中引入的新时间API使得无需使用外部库即可处理日期和时间。

在这个简短的教程中，我们将了解如何在不同版本的Java中更轻松地获取两个日期之间的所有日期。

## 2. 使用Java 7

在Java 7中，计算这一点的一种方法是使用Calendar实例。

首先，我们获取[不带时间的开始日期和结束日期](https://www.baeldung.com/java-date-without-time)，然后，我们将循环遍历这些日期，并使用add方法和Calendar.Date字段在每次迭代中增加一天，直到到达结束日期。

下面是使用Calendar实例的演示代码：

```java
public static List getDatesBetweenUsingJava7(Date startDate, Date endDate) {
    List datesInRange = new ArrayList<>();
    Calendar calendar = getCalendarWithoutTime(startDate);
    Calendar endCalendar = getCalendarWithoutTime(endDate);

    while (calendar.before(endCalendar)) {
        Date result = calendar.getTime();
        datesInRange.add(result);
        calendar.add(Calendar.DATE, 1);
    }

    return datesInRange;
}

private static Calendar getCalendarWithoutTime(Date date) {
    Calendar calendar = new GregorianCalendar();
    calendar.setTime(date);
    calendar.set(Calendar.HOUR, 0);
    calendar.set(Calendar.HOUR_OF_DAY, 0);
    calendar.set(Calendar.MINUTE, 0);
    calendar.set(Calendar.SECOND, 0);
    calendar.set(Calendar.MILLISECOND, 0);
    return calendar;
}
```

## 3. 使用Java 8

在Java 8中，我们现在可以创建一个连续的无限日期流，并只提取相关部分。遗憾的是，**当谓词匹配时，无法终止无限流**-这就是为什么我们需要计算这两天之间的天数，然后简单地对流进行limit()操作：

```java
public static List<LocalDate> getDatesBetweenUsingJava8(LocalDate startDate, LocalDate endDate) {
    long numOfDaysBetween = ChronoUnit.DAYS.between(startDate, endDate); 
    return IntStream.iterate(0, i -> i + 1)
        .limit(numOfDaysBetween)
        .mapToObj(i -> startDate.plusDays(i))
        .collect(Collectors.toList()); 
}
```

请注意，首先，我们可以使用between函数获取两个日期之间的天数差-与ChronoUnit枚举的DAYS常量相关联。

然后，我们创建一个整数流，表示自开始日期以来的天数。下一步，我们使用plusDays() API将整数转换为LocalDate对象。

最后，我们将所有内容收集到一个列表实例中。

## 4. 使用Java 9

最后，Java 9带来了专门的方法来计算这个：

```java
public static List<LocalDate> getDatesBetweenUsingJava9(LocalDate startDate, LocalDate endDate) {
    return startDate.datesUntil(endDate)
        .collect(Collectors.toList());
}
```

我们可以使用LocalDate类的专用datesUntil方法，通过一次方法调用获取两个日期之间的日期。datesUntil返回按顺序排列的日期流，从调用该方法的日期对象开始，到作为方法参数给出的日期为止。

## 5. 总结

在这篇简短的文章中，我们研究了如何使用不同版本的Java获取两个日期之间的所有日期。

Java 8版本中引入的Time API使对日期文字进行操作变得更容易，而在Java 9中，只需调用datesUntil即可完成。