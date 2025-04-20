---
layout: post
title:  解决DateTimeParseException：“Unable to obtain LocalDateTime from TemporalAccessor”
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

使用[java.time](https://www.baeldung.com/java-8-date-time-intro)包处理Java中的日期和时间非常高效，但有时我们可能会遇到DateTimeParseException错误，提示“无法从TemporalAccessor获取LocalDateTime”，此问题通常是由于预期的日期时间格式与实际输入不兼容而引起的。

本文解释了此异常发生的原因，探讨了其常见原因，并提供了预防和修复此异常的有效策略。

## 2. 理解异常

当Java的日期时间解析器无法从[TemporalAccessor](https://www.baeldung.com/java-temporalaccessor-localdate-conversion)中提取有效的LocalDateTime对象(例如LocalDate、ZonedDateTime或OffsetDateTime)时，就会出现“无法从TemporalAccessor获取LocalDateTime”异常，**根本原因通常是输入的字符串格式不正确或不完整**。

LocalDateTime需要日期和时间部分，如果输入字符串缺少必需部分或不符合预期格式，解析过程将失败，从而导致此异常。**我们通常认为Java可以自动推断缺失的时间值，但事实并非如此**。

让我们考虑以下示例，其中日期字符串被错误地解析为LocalDateTime：

```java
public static void main(String[] args) {
    String dateTimeStr = "20250327";
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyyMMdd");
    LocalDateTime localDateTime = LocalDateTime.parse(dateTimeStr, formatter);
}
```

执行时，此代码会引发以下异常：

```text
java.time.format.DateTimeParseException: Text '20250327' could not be parsed: Unable to obtain LocalDateTime from TemporalAccessor: {},ISO resolved to 2025-03-27 of type java.time.format.Parsed
```

发生错误的原因是LocalDateTime需要日期和时间，但输入仅包含日期。

## 3. 常见原因及解决方法

在深入探讨具体原因之前，务必认识到日期时间解析问题通常源于对输入数据结构的假设。java.time API对格式规则的执行非常严格，这意味着任何偏差(例如缺少时间部分、格式不正确或时区异常)都可能触发异常。

下面，我们将探讨此错误的最常见原因，并寻找有效处理它们的解决方案。

### 3.1 输入字符串中缺少时间部分

如果输入字符串仅包含日期，例如“2024-03-25”，但被解析为LocalDateTime，则解析会失败，因为LocalDateTime需要日期和时间两个部分，这会导致DateTimeParseException。

为了解决这个问题，我们可以将日期解析为LocalDate而不是LocalDateTime：

```java
LocalDate date = LocalDate.parse("2024-03-25", DateTimeFormatter.ISO_LOCAL_DATE);
```

或者，如果我们需要LocalDateTime，我们可以在输入字符串后附加一个默认时间值，例如“T00:00:00”：

```java
String dateTimeStr = "2024-03-25T00:00:00";
LocalDateTime dateTime = LocalDateTime.parse(dateTimeStr, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
```

### 3.2 将DayOfWeek解析为LocalDateTime

DayOfWeek枚举表示星期几(例如MONDAY、FRIDAY)，但不包含任何日期或时间信息。尝试将DayOfWeek值直接用作LocalDateTime会导致异常，因为LocalDateTime需要日期和时间。

如果我们需要特定工作日的完整LocalDateTime，我们可以确定该日的下一个发生时间并将其与选定的时间结合起来：

```java
DayOfWeek targetDay = DayOfWeek.FRIDAY;
LocalDate today = LocalDate.now();
LocalDate nextTargetDate = today.with(TemporalAdjusters.next(targetDay));

LocalTime time = LocalTime.of(14, 30);
LocalDateTime dateTime = LocalDateTime.of(nextTargetDate, time);
```

这种方法确保我们在将DayOfWeek值与时间组合形成有效的LocalDateTime之前，正确地将其与实际日期关联起来。

### 3.3 将LocalTime解析为LocalDateTime

当输入字符串仅包含时间(例如“14:30:00”)并被解析为LocalDateTime时，解析会失败，因为LocalDateTime需要日期和时间部分。LocalTime仅提供时间部分，因此将其解析为LocalDateTime会导致异常。

为了解决这个问题，我们将LocalTime与LocalDate结合起来形成一个完整的LocalDateTime：

```java
LocalDate date = LocalDate.of(2024, 3, 25);
LocalTime time = LocalTime.parse("14:30:00");
LocalDateTime dateTime = LocalDateTime.of(date, time);
```

### 3.4 将YearMonth解析为LocalDateTime

YearMonth类仅表示年份和月份，不包含任何具体的日期或时间信息。因此，尝试将YearMonth解析为LocalDateTime会失败，因为LocalDateTime需要完整的日期和时间。

为了解决这个问题，我们可以使用YearMonth类来执行只需要年份和月份的操作。或者，如果我们需要完整的LocalDateTime，我们可以将YearMonth与特定的日期和时间组合起来：

```java
YearMonth yearMonth = YearMonth.parse("2024-03", DateTimeFormatter.ofPattern("yyyy-MM"));
LocalDate date = yearMonth.atDay(1);
LocalTime time = LocalTime.of(14, 30);
LocalDateTime dateTime = LocalDateTime.of(date, time);
```

### 3.5 将MonthDay解析为LocalDateTime

MonthDay类仅表示月份和日期(例如“03-25”)，不包含年份或时间部分。将MonthDay解析为LocalDateTime会失败，因为LocalDateTime需要完整的日期和时间。

为了解决这个问题，如果只需要月份和日期，我们可以使用MonthDay。或者，如果我们需要LocalDateTime，我们可以将MonthDay与特定的年份和时间结合起来：

```java
MonthDay monthDay = MonthDay.parse("03-25", DateTimeFormatter.ofPattern("MM-dd"));
LocalDate date = LocalDate.of(2024, monthDay.getMonth(), monthDay.getDayOfMonth());
LocalTime time = LocalTime.of(14, 30);
LocalDateTime dateTime = LocalDateTime.of(date, time);
```

## 4. 总结

DateTimeParseException错误“无法从TemporalAccessor获取LocalDateTime”通常是由于缺少或格式不正确的日期时间信息，或时区处理不当而发生的。

**为了避免此错误，我们应该确保输入格式符合预期模式并使用适当的[Java日期时间类](https://www.baeldung.com/migrating-to-java-8-date-time-api)(LocalDate、LocalDateTime、ZonedDateTime或OffsetDateTime)**。

通过遵循这些最佳实践，可以有效地解析和管理日期时间值而不会出现错误。