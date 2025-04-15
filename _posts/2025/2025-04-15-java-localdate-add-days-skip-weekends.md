---
layout: post
title:  Java中LocalDate增加日期时跳过周末
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将简要介绍在Java 8中向LocalDate实例增加天数时跳过周末的算法。

我们还将通过算法**从LocalDate对象中减去天数，同时跳过周末**。

## 2. 增加天数

在这个方法中，我们不断地向LocalDate对象增加一天，直到增加所需的天数。增加每一天时，**我们会检查新的LocalDate实例的日期是星期六还是星期日**。

如果检查结果为true，则我们不会将当前已增加的天数计数器递增。但是，如果当前日期是工作日，则计数器会递增。

通过这种方式，我们不断增加天数，直到计数器等于应该增加的天数：

```java
public static LocalDate addDaysSkippingWeekends(LocalDate date, int days) {
    LocalDate result = date;
    int addedDays = 0;
    while (addedDays < days) {
        result = result.plusDays(1);
        if (!(result.getDayOfWeek() == DayOfWeek.SATURDAY || result.getDayOfWeek() == DayOfWeek.SUNDAY)) {
            ++addedDays;
        }
    }
    return result;
}
```

在上面的代码中，我们使用LocalDate对象的plusDays()方法向结果对象增加日期。仅当日期是工作日时，我们才会增加addedDays变量的值。当变量addedDays等于days变量时，我们停止向结果LocalDate对象增加日期。

## 3. 减去天数

类似地，我们可以使用minusDays()方法从LocalDate对象中减去天数，直到减去所需的天数。

为了实现这一点，**我们将保留一个计数器来记录减去的天数，只有当结果日期是工作日时，该计数器才会递增**：

```java
public static LocalDate subtractDaysSkippingWeekends(LocalDate date, int days) {
    LocalDate result = date;
    int subtractedDays = 0;
    while (subtractedDays < days) {
        result = result.minusDays(1);
        if (!(result.getDayOfWeek() == DayOfWeek.SATURDAY || result.getDayOfWeek() == DayOfWeek.SUNDAY)) {
            ++subtractedDays;
        }
    }
    return result;
}
```

从上面的实现中，我们可以看到，只有当LocalDate对象的结果为工作日时，subtractedDays才会递增。使用while循环，我们不断减去天数，直到subtractedDays等于days变量。

## 4. 总结

在这篇简短的文章中，我们研究了在LocalDate对象中跳过周末增加和减去天数的算法。此外，我们还研究了这些算法在Java中的实现。