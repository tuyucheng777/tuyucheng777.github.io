---
layout: post
title:  用Java计算两个日期之间的工作日数
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将学习Java中两种不同的方法来计算两个日期之间有多少个工作日。我们将介绍一个使用[Stream](https://www.baeldung.com/java-streams)的可读性版本，以及一个可读性稍差但效率更高的版本(完全不循环)。

## 2. 使用Stream进行全搜索

首先，让我们看看如何使用Stream来实现这一点。**计划是循环遍历两个日期之间的每一天，并计算出其中的星期几**：

```java
long getWorkingDaysWithStream(LocalDate start, LocalDate end){
    return start.datesUntil(end)
        .map(LocalDate::getDayOfWeek)
        .filter(day -> !Arrays.asList(DayOfWeek.SATURDAY, DayOfWeek.SUNDAY).contains(day))
        .count();
}
```

首先，我们使用了[LocalDate](https://www.baeldung.com/java-8-date-time-intro)的datesUntil()方法，此方法返回一个Stream，其中包含从开始日期(含)到结束日期(不含)的所有日期。

接下来，我们使用map()和LocalDate的getDayOfWeek()将每个日期转换为星期几。例如，这会将2023年10月1日更改为星期三。

接下来，我们通过与DaysOfWeek[枚举](https://www.baeldung.com/a-guide-to-java-enums)进行比对，筛选出所有周末。最后，我们可以计算出剩下的天数，因为我们知道这些天都是工作日。

**这种方法并非最快捷，因为我们每天都要查看**。然而，它很容易理解，并且可以根据需要轻松添加额外的检查或处理。

## 3. 高效搜索，无需循环

**另一个选择是，不循环遍历所有日期，而是应用我们已知的星期几规则**。这里我们需要几个步骤，以及一些需要处理的边缘情况。

### 3.1 设置初始日期

首先，定义我们的方法签名，它与之前的非常相似：

```java
long getWorkingDaysWithoutStream(LocalDate start, LocalDate end)
```

**处理这些日期的第一步是在开始和结束时排除任何周末**。因此，对于开始日期，如果是周末，我们将取下周一，我们还会使用布尔值来跟踪执行此操作的事实：

```java
boolean startOnWeekend = false;
if(start.getDayOfWeek().getValue() > 5){
    start = start.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
    startOnWeekend = true;
}
```

我们在这里使用了[TemporalAdjusters](https://www.baeldung.com/java-temporal-adjuster)类，特别是它的next()方法，它让我们跳转到下一个指定的日期。

然后，我们可以对结束日期执行相同的操作——如果是周末，则取上一个星期五。这次，我们将使用TemporalAdjusters.previous()来定位到给定日期之前我们想要的日期的第一次出现：

```java
boolean endOnWeekend = false;
if(end.getDayOfWeek().getValue() > 5){
    end = end.with(TemporalAdjusters.previous(DayOfWeek.FRIDAY));
    endOnWeekend = true;
}
```

### 3.2 考虑边缘情况

这已经给我们带来了一个潜在的边缘情况：如果我们从周六开始，周日结束。在这种情况下，我们的开始日期将是周一，结束日期将是之前的周五。开始时间晚于结束时间是没有意义的，所以我们可以通过快速检查来涵盖这个潜在的用例：

```java
if(start.isAfter(end)){
    return 0;
}
```

**我们还需要考虑另一种特殊情况，这就是为什么我们要记录周末的开始和结束**。这是可选的，取决于我们想要如何计算天数。例如，如果我们计算同一周的星期二和星期五之间的天数，我们会认为它们之间有三天。

我们也假设周六和下一个周六之间有五个工作日，但是，如果我们像这里一样将开始日期和结束日期分别移到周一和周五，那么就算作四天。因此，为了抵消这种情况，我们可以根据需要增加一天：

```java
long addValue = startOnWeekend || endOnWeekend ? 1 : 0;
```

### 3.3 最终计算

现在，我们可以计算开始和结束之间的总周数。为此，**我们将使用ChronoUnit的between()方法，此方法以指定的单位(在本例中为WEEKS)计算两个Temporal对象之间的时间**：

```java
long weeks = ChronoUnit.WEEKS.between(start, end);
```

**最后，我们可以使用迄今为止收集到的所有信息来获得工作日数量的最终值**：

```java
return ( weeks * 5 ) + ( end.getDayOfWeek().getValue() - start.getDayOfWeek().getValue() ) + addValue;
```

这里的步骤是首先将周数乘以每周的工作日数，我们还没有考虑非整周的情况，所以我们添加了一周开始日和一周结束日之间的额外天数。最后，我们添加了在周末开始或结束的调整。

## 4. 总结

在本文中，我们研究了计算两个日期之间工作日数的两种方法。

首先，我们了解了如何使用Stream来分别检查每一天的数据，这种方法简单易读，但效率较低。

第二种选择是应用我们已知的星期几规则来计算，无需循环；这种方法效率更高，但可读性和可维护性会受到影响。