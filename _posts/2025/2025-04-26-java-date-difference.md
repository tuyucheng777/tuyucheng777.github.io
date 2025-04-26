---
layout: post
title:  Java中两个日期之间的差异
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本快速教程中，我们将探索在Java中计算两个日期之间差异的多种方法。

## 2. 核心Java

### 2.1 使用 java.util.Date查找天数差异

让我们首先使用核心Java API进行计算并确定两个日期之间的天数：

```java
@Test
public void givenTwoDatesBeforeJava8_whenDifferentiating_thenWeGetSix() throws ParseException {
    SimpleDateFormat sdf = new SimpleDateFormat("MM/dd/yyyy", Locale.ENGLISH);
    Date firstDate = sdf.parse("06/24/2017");
    Date secondDate = sdf.parse("06/30/2017");

    long diffInMillies = Math.abs(secondDate.getTime() - firstDate.getTime());
    long diff = TimeUnit.DAYS.convert(diffInMillies, TimeUnit.MILLISECONDS);

    assertEquals(6, diff);
}
```

### 2.2 使用java.time.temporal.ChronoUnit查找差异

Java 8中的时间API使用[TemporalUnit](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/temporal/Temporal.html) 接口表示日期时间单位，例如秒或天。

**每个单元都提供一个名为[between()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/temporal/TemporalUnit.html#between(java.time.temporal.Temporal,java.time.temporal.Temporal))的方法的实现，以根据特定单元计算两个时间对象之间的时间量**。

例如，计算两个LocalDateTime实例之间的秒数：

```java
@Test
public void givenTwoDateTimesInJava8_whenDifferentiatingInSeconds_thenWeGetTen() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime tenSecondsLater = now.plusSeconds(10);

    long diff = ChronoUnit.SECONDS.between(now, tenSecondsLater);

    assertEquals(10, diff);
}
```

[ChronoUnit](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/temporal/ChronoUnit.html)通过实现TemporalUnit接口提供了一组具体的时间单位。**强烈建议静态导入ChronoUnit枚举值，以提高可读性**：

```java
import static java.time.temporal.ChronoUnit.SECONDS;

// omitted
long diff = SECONDS.between(now, tenSecondsLater);
```

此外，我们可以将任意两个兼容的时间对象传递给between方法，甚至是ZonedDateTime。

ZonedDateTime的优点在于，即使设置为不同的时区，计算也能有效：

```java
@Test
public void givenTwoZonedDateTimesInJava8_whenDifferentiating_thenWeGetSix() {
    LocalDateTime ldt = LocalDateTime.now();
    ZonedDateTime now = ldt.atZone(ZoneId.of("America/Montreal"));
    ZonedDateTime sixMinutesBehind = now
        .withZoneSameInstant(ZoneId.of("Asia/Singapore"))
        .minusMinutes(6);
    
    long diff = ChronoUnit.MINUTES.between(sixMinutesBehind, now);
    
    assertEquals(6, diff);
}
```

### 2.3 使用Temporal#until()

**任何[Temporal](https://docs.oracle.com/en/java/javase/21/docs/api//java.base/java/time/temporal/Temporal.html)对象(例如LocalDate或 ZonedDateTime)都提供了一种[until](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/temporal/Temporal.html#until(java.time.temporal.Temporal,java.time.temporal.TemporalUnit))方法来根据指定的单位计算到另一个Temporal的时间量**：

```java
@Test
public void givenTwoDateTimesInJava8_whenDifferentiatingInSecondsUsingUntil_thenWeGetTen() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime tenSecondsLater = now.plusSeconds(10);

    long diff = now.until(tenSecondsLater, ChronoUnit.SECONDS);

    assertEquals(10, diff);
}
```

Temporal#until和TemporalUnit#between是实现相同功能的两个不同API。

### 2.4 使用java.time.Duration和java.time.Period

在Java 8中，Time API引入了两个新类：[Duration和Period](https://www.baeldung.com/java-8-date-time-intro#period)。

**如果我们想要计算两个日期时间在基于时间(小时、分钟或秒)的时间段内的差值，我们可以使用Duration类**：

```java
@Test
public void givenTwoDateTimesInJava8_whenDifferentiating_thenWeGetSix() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime sixMinutesBehind = now.minusMinutes(6);

    Duration duration = Duration.between(now, sixMinutesBehind);
    long diff = Math.abs(duration.toMinutes());

    assertEquals(6, diff);
}
```

然而，**如果我们尝试使用Period类来表示两个日期之间的差异，我们应该警惕一个陷阱**。

一个例子可以快速解释这个陷阱。

让我们使用Period类来计算两个日期之间有多少天：

```java
@Test
public void givenTwoDatesInJava8_whenUsingPeriodGetDays_thenWorks() {
    LocalDate aDate = LocalDate.of(2020, 9, 11);
    LocalDate sixDaysBehind = aDate.minusDays(6);

    Period period = Period.between(aDate, sixDaysBehind);
    int diff = Math.abs(period.getDays());

    assertEquals(6, diff);
}
```

如果我们运行上面的测试，它会通过，我们可能会认为Period类对于解决我们的问题很方便。到目前为止，一切顺利。

如果这种方法在相差六天的情况下仍然有效，我们毫不怀疑它在60天内也同样有效。

那么让我们将上面测试中的6改为60，看看会发生什么：

```java
@Test
public void givenTwoDatesInJava8_whenUsingPeriodGetDays_thenDoesNotWork() {
    LocalDate aDate = LocalDate.of(2020, 9, 11);
    LocalDate sixtyDaysBehind = aDate.minusDays(60);

    Period period = Period.between(aDate, sixtyDaysBehind);
    int diff = Math.abs(period.getDays());

    assertEquals(60, diff);
}
```

现在如果我们再次运行测试，我们将看到：

```text
java.lang.AssertionError:
Expected :60
Actual   :29
```

哎呀！为什么Period类报告的差异是29天？

**这是因为Period类表示基于日期的时间量，格式为“x年y月z天”。当我们调用它的getDays()方法时，它只返回“z天”部分**。

因此，上面测试中的period对象保存的值是“0年 1个月29天”：

```java
@Test
public void givenTwoDatesInJava8_whenUsingPeriod_thenWeGet0Year1Month29Days() {
    LocalDate aDate = LocalDate.of(2020, 9, 11);
    LocalDate sixtyDaysBehind = aDate.minusDays(60);
    Period period = Period.between(aDate, sixtyDaysBehind);
    int years = Math.abs(period.getYears());
    int months = Math.abs(period.getMonths());
    int days = Math.abs(period.getDays());
    assertArrayEquals(new int[] { 0, 1, 29 }, new int[] { years, months, days });
}
```

**如果我们想使用Java 8的时间API计算天数差异，ChronoUnit.DAYS.between()方法是最直接的方法**。

## 3. 外部库

### 3.1 JodaTime

我们还可以使用JodaTime做一个相对简单的实现：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.13.1</version>
</dependency>
```

[Joda-time](https://mvnrepository.com/search?q=joda-time)的最新版本可从Maven Central获得。

让我们看一下LocalDate案例：

```java
@Test
public void givenTwoDatesInJodaTime_whenDifferentiating_thenWeGetSix() {
    org.joda.time.LocalDate now = org.joda.time.LocalDate.now();
    org.joda.time.LocalDate sixDaysBehind = now.minusDays(6);

    long diff = Math.abs(Days.daysBetween(now, sixDaysBehind).getDays());
    assertEquals(6, diff);
}
```

类似地，我们可以使用LocalDateTime：

```java
@Test
public void givenTwoDateTimesInJodaTime_whenDifferentiating_thenWeGetSix() {
    org.joda.time.LocalDateTime now = org.joda.time.LocalDateTime.now();
    org.joda.time.LocalDateTime sixMinutesBehind = now.minusMinutes(6);

    long diff = Math.abs(Minutes.minutesBetween(now, sixMinutesBehind).getMinutes());
    assertEquals(6, diff);
}
```

### 3.2 Date4J

Date4j还提供了一个简单的实现-注意，在这种情况下，我们需要明确提供一个TimeZone。

让我们从Maven依赖开始：

```xml
<dependency>
    <groupId>com.darwinsys</groupId>
    <artifactId>hirondelle-date4j</artifactId>
    <version>1.5.1</version>
</dependency>
```

以下是使用标准DateTime的快速测试：

```java
@Test
public void givenTwoDatesInDate4j_whenDifferentiating_thenWeGetSix() {
    DateTime now = DateTime.now(TimeZone.getDefault());
    DateTime sixDaysBehind = now.minusDays(6);
 
    long diff = Math.abs(now.numDaysFrom(sixDaysBehind));

    assertEquals(6, diff);
}
```

## 4. 获取两个日期之间的周数

我们还可以使用日期时间API之一获取两个日期之间的周数。

### 4.1 使用Joda-Time中的DateTime

Joda-Time中的org.joda.time.Weeks类存储了一个不可变的周期，该周期代表特定的周数。我们可以使用getWeeks()方法查询两个日期之间的差值(以周为单位) 。

让我们找出两个[org.joda.time.DateTime](https://www.baeldung.com/joda-time#2-custom-date-and-time)值之间的周数：

```java
@Test
public void givenTwoDateTimesInJodaTime_whenComputingDistanceInWeeks_thenFindIntegerWeeks() {
    DateTime dateTime1 = new DateTime(2024, 1, 17, 15, 50, 30);
    DateTime dateTime2 = new DateTime(2024, 6, 3, 10, 20, 55);
    int weeksDiff = Weeks.weeksBetween(dateTime1, dateTime2).getWeeks();

    assertEquals(19, weeksDiff);
}
```

JUnit测试应该通过，因为两个DateTime值的周数差为19。

但是，Weeks只能存储周数，无法存储天数、小时数或分钟数，两个示例DateTime值都剩余了四天。

我们可以通过使用org.joda.time.Days类来实现更高的精度：

```java
@Test
public void givenTwoDateTimesInJodaTime_whenComputingDistanceInDecimalWeeks_thenFindDecimalWeeks() {
    DateTime dateTime1 = new DateTime(2024, 1, 17, 15, 50, 30);
    DateTime dateTime2 = new DateTime(2024, 6, 3, 10, 20, 55);

    int days = Days.daysBetween(dateTime1, dateTime2).getDays();
    float weeksDiff=(float) (days/7.0);

    assertEquals(19.571428, weeksDiff, 0.001);
}
```

请注意，我们将表示天数差异的[int](https://www.baeldung.com/java-integer-division-float-result)值转换为float或double数，以获得十进制级别的精度。

### 4.2 使用LocalDate

[java.time.LocalDate](https://www.baeldung.com/joda-time#1-current-date-and-time)类是一个不可变的日期时间对象，它表示ISO-8601日历系统中没有时区的日期，例如2007-12-03。

使用LocalDate实例，我们可以使用ChronoUnit.WEEKS.between()方法确定周数差异：

```java
@Test
public void givenTwoLocalDatesInJava8_whenUsingChronoUnitWeeksBetween_thenFindIntegerWeeks() {
    LocalDate startLocalDate = LocalDate.of(2024, 01, 10);
    LocalDate endLocalDate = LocalDate.of(2024, 11, 15);

    long weeksDiff = ChronoUnit.WEEKS.between(startLocalDate, endLocalDate);

    assertEquals(44, weeksDiff);
}
```

与之前的Weeks#weeksBetween一样，这会截断任何多余的天数。

### 4.3 使用ZonedDateTime

[java.time.ZonedDateTime](https://www.baeldung.com/java-zoneddatetime-offsetdatetime)类是不可变的日期时间类，它表示ISO-8601日历系统中带有时区的日期，例如2007-12-03T10:15:30+01:00。

再次，我们可以使用ChronoUnit.WEEKS.between()方法确定周数差异：

```java
@Test
public void givenTwoZonedDateTimesInJava8_whenUsingChronoUnitWeeksBetween_thenFindIntegerWeeks() {
    ZonedDateTime startDateTime = ZonedDateTime.parse("2022-02-01T00:00:00Z[UTC]");
    ZonedDateTime endDateTime = ZonedDateTime.parse("2022-10-31T23:59:59Z[UTC]");

    long weeksDiff = ChronoUnit.WEEKS.between(startDateTime, endDateTime);

    assertEquals(38, weeksDiff);
}
```

这也会截断任何额外的天数。

## 5. 总结

在本文中，我们演示了几种计算日期(包含和不包含时间)之间差值的方法，既可以使用纯Java实现，也可以使用外部库实现。此外，我们还计算了以天为单位和以周为单位的日期之间的差值。