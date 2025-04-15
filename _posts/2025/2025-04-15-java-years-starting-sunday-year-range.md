---
layout: post
title:  确定给定年份范围内以星期日开始的所有年份
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

确定所有以星期日开始的年份看似一个简单的需求，然而，它在各种实际场景或业务用例中可能非常相关。特定的日期和日历结构通常会影响运营、事件或日程安排。例如，在节假日或宗教活动安排，或工资单和工作日程规划中，可能会出现这样的需求。

在本教程中，我们将介绍三种方法，用于查找给定范围内所有以星期日开始的年份。首先，我们将介绍一种使用[Date和Calendar](https://www.baeldung.com/java-8-date-time-intro)的解决方案，然后介绍如何使用现代java.time API。最后，我们将包含一个使用[Spliterator](https://www.baeldung.com/java-spliterator)<LocalDate\>的优化示例。

## 2. 遗留解决方案

首先，**我们将使用旧版Calendar类来检查给定范围内每年的1月1日是否是星期日**： 

```java
public class FindSundayStartYearsLegacy {

    public static List<Integer> getYearsStartingOnSunday(int startYear, int endYear) {
        List<Integer> years = new ArrayList<>();

        for (int year = startYear; year <= endYear; year++) {
            Calendar calendar = new GregorianCalendar(year, Calendar.JANUARY, 1);
            if (calendar.get(Calendar.DAY_OF_WEEK) == Calendar.SUNDAY) {
                years.add(year);
            }
        }
        return years;
    }
}
```

首先，我们为每年创建一个Calendar对象，并将日期设置为1月1日。然后，我们使用calendar.get(Calendar.DAY_OF_WEEK)方法检查该日期是否为星期日。我们将使用2000到2025年的年份范围来测试FindSundayStartYearsLegacy的实现，因此期望结果包含以星期日开始的四个年份：2006、2012、2017和2023：

```java
@Test
public void givenYearRange_whenCheckingStartDayLegacy_thenReturnYearsStartingOnSunday() {
    List<Integer> expected = List.of(2006, 2012, 2017, 2023);
    List<Integer> result = FindSundayStartYearsLegacy.getYearsStartingOnSunday(2000, 2025);
    assertEquals(expected, result);
}
```

## 3. 新的API解决方案

**让我们使用Java 8中引入的java.time包来解决同样的问题**，我们将使用LocalDate来检查每年的1月1日是否是星期日：

```java
public class FindSundayStartYearsTimeApi {

    public static List<Integer> getYearsStartingOnSunday(int startYear, int endYear) {
        List<Integer> years = new ArrayList<>();

        for (int year = startYear; year <= endYear; year++) {
            LocalDate firstDayOfYear = LocalDate.of(year, 1, 1);
            if (firstDayOfYear.getDayOfWeek() == DayOfWeek.SUNDAY) {
                years.add(year);
            }
        }
        return years;
    }
}
```

这里我们使用LocalDate.of(year, 1, 1)来表示每年的1月1日，然后，我们使用getDayOfWeek()方法检查该日期是否为星期日。接下来，我们使用FindSundayStartYearsTimeApi类中的java.time API验证解决方案，输入和预期结果与上一个测试相同：

```java
@Test 
public void givenYearRange_whenCheckingStartDayTimeApi_thenReturnYearsStartingOnSunday() {
    List<Integer> expected = List.of(2006, 2012, 2017, 2023);
    List<Integer> result = FindSundayStartYearsTimeApi.getYearsStartingOnSunday(2000, 2025);
    assertEquals(expected, result);
}
```

## 4. 使用Stream的增强功能

**让我们看看如何使用Spliterator<LocalDate\>来遍历年份并过滤掉从星期日开始的年份**，Spliterator对于高效遍历大数据范围非常有用：

```java
public class FindSundayStartYearsSpliterator {
    public static List<Integer> getYearsStartingOnSunday(int startYear, int endYear) {
        List<Integer> years = new ArrayList<>();
        Spliterator<LocalDate> spliterator = Spliterators.spliteratorUnknownSize(
                Stream.iterate(LocalDate.of(startYear, 1, 1), date -> date.plus(1, ChronoUnit.YEARS))
                        .limit(endYear - startYear + 1)
                        .iterator(), Spliterator.ORDERED);

        Stream<LocalDate> stream = StreamSupport.stream(spliterator, false);
        stream.filter(date -> date.getDayOfWeek() == DayOfWeek.SUNDAY)
                .forEach(date -> years.add(date.getYear()));
        return years;
    }
}
```

这里我们使用Stream.iterator()创建了一个Spliterator<LocalDate\>对象，用于迭代给定范围内每年的第一天。接下来，StreamSupport.stream()方法将Spliterator转换为Stream<LocalDate\>对象。此外，我们使用filter() 函数检查每年的第一天是否为星期日。最后，我们将有效条目填充到数组中，并在最后返回。现在，我们将在FindSundayStartYearsSpliterator类中测试基于Streams的实现：

```java
@Test
public void givenYearRange_whenCheckingStartDaySpliterator_thenReturnYearsStartingOnSunday() {
    List<Integer> expected = List.of(2006, 2012, 2017, 2023);
    List<Integer> result = FindSundayStartYearsSpliterator.getYearsStartingOnSunday(2000, 2025);
    assertEquals(expected, result);
}
```

## 5. 总结

在本文中，我们研究了三种在给定范围内查找从星期日开始的年份的选项。尽管Date和Calendar可用，但它们存在诸如可变性、笨重的API和弃用的方法等问题。这种遗留解决方案很简单，但不是新开发的最佳实践。通常，java.time包提供了更清晰、更强大的API来处理日期和时间。它类型安全、不可变，并且比Date和Calendar更易于使用。因此，将Spliterator与Stream结合使用是一种处理大量数据的有效方法，允许并行化和惰性求值。java.time方法是使用Java处理日期和时间的推荐方法，Spliterator解决方案增加了灵活性和性能优势。