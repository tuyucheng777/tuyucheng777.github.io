---
layout: post
title:  DateTimeFormatter指南
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将回顾Java 8 DateTimeFormatter类及其格式化模式，我们还将讨论此类的可能用例。

我们可以使用DateTimeFormatter以预定义或用户定义的模式统一格式化应用程序中的日期和时间。

## 2. 具有预定义实例的DateTimeFormatter

**DateTimeFormatter附带多种符合ISO和 RFC标准的[预定义日期/时间格式](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/format/DateTimeFormatter.html)**，例如，我们可以使用ISO_LOCAL_DATE实例来解析诸如“2018-03-09”这样的日期：

```java
DateTimeFormatter.ISO_LOCAL_DATE.format(LocalDate.of(2018, 3, 9));
```

要解析带有偏移量的日期，我们可以使用ISO_OFFSET_DATE来获取类似“2018-03-09-03:00”的输出：

```java
DateTimeFormatter.ISO_OFFSET_DATE.format(LocalDate.of(2018, 3, 9).atStartOfDay(ZoneId.of("UTC-3")));
```

**DateTimeFormatter类的大多数预定义实例都侧重于ISO-8601标准**，ISO-8601是日期和时间格式的国际标准。

但是，有一个不同的预定义实例可以解析IETF[发布](https://tools.ietf.org/html/rfc1123.html)的RFC-1123《Internet主机要求》：

```java
DateTimeFormatter.RFC_1123_DATE_TIME.format(LocalDate.of(2018, 3, 9).atStartOfDay(ZoneId.of("UTC-3")));
```

此代码片段生成“Fri, 9 Mar 2018 00:00:00 -0300”。

有时我们需要将接收到的日期转换为已知格式的字符串。为此，我们可以使用parse()方法：

```java
LocalDate.from(DateTimeFormatter.ISO_LOCAL_DATE.parse("2018-03-09")).plusDays(3);
```

该代码片段的结果是2018年3月12日的LocalDate表示。

## 3. 带有FormatStyle的DateTimeFormatter

有时我们可能希望以人类可读的方式打印日期。

在这种情况下，我们可以使用java.time.format.FormatStyle枚举(FULL、LONG、MEDIUM、SHORT)值与我们的DateTimeFormatter：

```java
LocalDate anotherSummerDay = LocalDate.of(2016, 8, 23);
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).format(anotherSummerDay));
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG).format(anotherSummerDay));
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM).format(anotherSummerDay));
System.out.println(DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT).format(anotherSummerDay));
```

同一日期的不同格式样式的输出如下：

```text
Tuesday, August 23, 2016
August 23, 2016
Aug 23, 2016
8/23/16
```

我们还可以使用预定义的日期和时间格式，要使用FormatStyle格式，必须使用ZonedDateTime实例，否则会抛出DateTimeException异常：

```java
LocalDate anotherSummerDay = LocalDate.of(2016, 8, 23);
LocalTime anotherTime = LocalTime.of(13, 12, 45);
ZonedDateTime zonedDateTime = ZonedDateTime.of(anotherSummerDay, anotherTime, ZoneId.of("Europe/Helsinki"));
System.out.println(
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL)
    .format(zonedDateTime));
System.out.println(
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG)
    .format(zonedDateTime));
System.out.println(
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM)
    .format(zonedDateTime));
System.out.println(
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT)
    .format(zonedDateTime));
```

注意，这次我们使用了DateTimeFormatter的ofLocalizedDateTime()方法。

我们得到的输出是：

```text
Tuesday, August 23, 2016 1:12:45 PM EEST
August 23, 2016 1:12:45 PM EEST
Aug 23, 2016 1:12:45 PM
8/23/16 1:12 PM
```

例如，我们还可以使用FormatStyle来解析日期时间字符串，将其转换为ZonedDateTime。

然后我们可以使用解析的值来操作日期和时间变量：

```java
ZonedDateTime dateTime = ZonedDateTime.from(DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL)
    .parse("Tuesday, August 23, 2016 1:12:45 PM EET"));
System.out.println(dateTime.plusHours(9));
```

此代码片段的输出为“2016-08-23T22:12:45+03:00[Europe/Bucharest\]”，请注意，时间已更改为“22:12:45”。

## 4. 自定义格式的DateTimeFormatter

**预定义和内置的格式化程序和样式可以涵盖很多情况**。然而，有时我们需要以不同的方式格式化日期和时间。这时，自定义格式模式就派上用场了。

### 4.1 日期的DateTimeFormatter

假设我们想要使用常规欧洲格式(例如2018.12.31)来呈现java.time.LocalDate对象，为此，我们可以调用工厂方法DateTimeFormatter.ofPattern(“dd.MM.yyyy”)。

这将创建一个适当的DateTimeFormatter实例，我们可以使用它来格式化我们的日期：

```java
String europeanDatePattern = "dd.MM.yyyy";
DateTimeFormatter europeanDateFormatter = DateTimeFormatter.ofPattern(europeanDatePattern);
System.out.println(europeanDateFormatter.format(LocalDate.of(2016, 7, 31)));
```

该代码片段的输出为“31.07.2016”。

我们可以使用许多不同的模式字母来创建适合我们需要的日期格式：

```text
  Symbol  Meaning                     Presentation      Examples
  ------  -------                     ------------      -------
   u       year                        year              2004; 04
   y       year-of-era                 year              2004; 04
   M/L     month-of-year               number/text       7; 07; Jul; July; J
   d       day-of-month                number            10
```

这是[Java官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/format/DateTimeFormatter.html)中有关DateTimeFormatter类的摘录。

**模式格式中的字母数量很重要**。

如果我们使用双字母模式表示月份，则会得到两位数的月份表示。如果月份数小于10，则会用零填充。如果不需要上述零填充，我们可以使用单字母模式“M”，这样一月就会显示为“1”。

如果我们恰好使用四个字母的模式表示月份“MMMM”，那么我们将得到一个“完整形式”的表示，在我们的例子中，它将是“July”。五个字母的模式“MMMMM”将使格式化程序使用“窄形式”，在我们的例子中，将使用“J”。

同样，自定义格式模式也可用于解析包含日期的字符串：

```java
DateTimeFormatter europeanDateFormatter = DateTimeFormatter.ofPattern("dd.MM.yyyy");
System.out.println(LocalDate.from(europeanDateFormatter.parse("15.08.2014")).isLeapYear());
```

此代码片段检查日期“15.08.2014”是否为闰年，结果不是。

### 4.2 时间的DateTimeFormatter

还有一些模式字母可用于时间模式：

```text
  Symbol  Meaning                     Presentation      Examples
  ------  -------                     ------------      -------
   H       hour-of-day (0-23)          number            0
   m       minute-of-hour              number            30
   s       second-of-minute            number            55
   S       fraction-of-second          fraction          978
   n       nano-of-second              number            987654321
```

使用DateTimeFormatter格式化java.time.LocalTime实例非常简单，假设我们想显示以冒号分隔的时间(小时、分钟和秒)：

```java
String timeColonPattern = "HH:mm:ss";
DateTimeFormatter timeColonFormatter = DateTimeFormatter.ofPattern(timeColonPattern);
LocalTime colonTime = LocalTime.of(17, 35, 50);
System.out.println(timeColonFormatter.format(colonTime));
```

这将生成输出“17:35:50”。

如果我们想在输出中添加毫秒，我们应该在模式中添加“SSS”：

```java
String timeColonPattern = "HH:mm:ss SSS";
DateTimeFormatter timeColonFormatter = DateTimeFormatter.ofPattern(timeColonPattern);
LocalTime colonTime = LocalTime.of(17, 35, 50).plus(329, ChronoUnit.MILLIS);
System.out.println(timeColonFormatter.format(colonTime));
```

这给了我们输出“17:35:50 329”。

请注意，“HH”是一个小时模式，其输出范围为0-23。当我们想要显示AM/PM时，应该使用小写“hh”表示小时，并添加“a”模式：

```java
String timeColonPattern = "hh:mm:ss a";
DateTimeFormatter timeColonFormatter = DateTimeFormatter.ofPattern(timeColonPattern);
LocalTime colonTime = LocalTime.of(17, 35, 50);
System.out.println(timeColonFormatter.format(colonTime));
```

生成的输出是“05:35:50 PM”。

我们可能想用我们的自定义格式化程序解析时间字符串并检查它是否在中午之前：

```java
DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("hh:mm:ss a");
System.out.println(LocalTime.from(timeFormatter.parse("12:25:30 AM")).isBefore(LocalTime.NOON));
```

最后一段代码的输出表明给定的时间实际上是在中午之前。

### 4.3 用于时区的DateTimeFormatter

**我们经常需要查看某个特定日期时间变量的时区**，例如，如果我们使用基于纽约的日期时间(UTC-4)，我们可以使用“z”模式字母作为时区名称：

```java
String newYorkDateTimePattern = "dd.MM.yyyy HH:mm z";
DateTimeFormatter newYorkDateFormatter = DateTimeFormatter.ofPattern(newYorkDateTimePattern);
LocalDateTime summerDay = LocalDateTime.of(2016, 7, 31, 14, 15);
System.out.println(newYorkDateFormatter.format(ZonedDateTime.of(summerDay, ZoneId.of("UTC-4"))));
```

这将生成输出“31.07.2016 14:15 UTC-04:00”。

我们可以像之前一样解析带有时区的日期时间字符串：

```java
DateTimeFormatter zonedFormatter = DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm z");
System.out.println(ZonedDateTime.from(zonedFormatter.parse("31.07.2016 14:15 GMT+02:00")).getOffset().getTotalSeconds());
```

正如我们预期的那样，此代码的输出是“7200”秒，或2小时。

我们必须确保向parse()方法提供正确的日期时间字符串。如果我们在上一段代码中将不带时区的“31.07.2016 14:15”传递给zonedFormatter，就会抛出DateTimeParseException异常。

### 4.4 使用区域设置的DateTimeFormatter

不仅可以使用特定时区获取正确的时间，还可以生成使用特定区域设置格式的日期格式，让我们用美国区域设置来检查一下：

```java
LocalDate date = LocalDate.of(2023, 9, 18);
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMM dd, yy: EEE").withLocale(Locale.US);
String formattedDate = date.format(formatter);
```

在这里，我们期望看到以下输出：

```text
Sep 18, 23: Mon
```

让我们尝试使用更明确的格式：

```java
LocalDate date = LocalDate.of(2023, 9, 18);
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMMM dd, yyyy: EEEE").withLocale(Locale.US);
String formattedDate = date.format(formatter);
```

这将产生以下结果：

```text
September 18, 2023: Monday
```

现在，我们可以更改本地语言，这样就能生成格式正确的日期了。在本例中，我们将使用韩国语言。代码与上面的代码片段类似，为了简洁起见，这里不再赘述。在第一种情况下，我们将得到以下输出：

```text
9월 18, 23: 월
```

这是一个更明确的：

```text
9월 18, 2023: 월요일
```

这是为任何本地生成正确格式的非常方便的方法，并且Java提供了这个开箱即用的机会。

### 4.5 即时日期时间格式化程序

DateTimeFormatter附带一个很棒的ISO即时格式化程序，名为ISO_INSTANT，顾名思义，此格式化程序提供了一种便捷的方式来格式化或解析UTC时刻。

根据[官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_INSTANT)，**如果不指定时区，则无法将时刻格式化为日期或时间**。因此，尝试在LocalDateTime或LocalDate对象上使用ISO_INSTANT将导致异常：

```java
@Test(expected = UnsupportedTemporalTypeException.class)
public void shouldExpectAnExceptionIfInputIsLocalDateTime() {
    DateTimeFormatter.ISO_INSTANT.format(LocalDateTime.now());
}
```

但是，我们可以使用ISO_INSTANT来格式化ZonedDateTime实例而不会出现任何问题：

```java
@Test
public void shouldPrintFormattedZonedDateTime() {
    ZonedDateTime zonedDateTime = ZonedDateTime.of(2021, 02, 15, 0, 0, 0, 0, ZoneId.of("Europe/Paris"));
    String formattedZonedDateTime = DateTimeFormatter.ISO_INSTANT.format(zonedDateTime);
    
    Assert.assertEquals("2021-02-14T23:00:00Z", formattedZonedDateTime);
}
```

可以看到，我们创建了“Europe/Paris”时区的ZonedDateTime。然而，格式化的结果却是UTC时间。

类似地，**当解析为ZonedDateTime时，我们需要指定时区**：

```java
@Test
public void shouldParseZonedDateTime() {
    DateTimeFormatter formatter = DateTimeFormatter.ISO_INSTANT.withZone(ZoneId.systemDefault());
    ZonedDateTime zonedDateTime = ZonedDateTime.parse("2021-10-01T05:06:20Z", formatter);
    
    Assert.assertEquals("2021-10-01T05:06:20Z", DateTimeFormatter.ISO_INSTANT.format(zonedDateTime));
}
```

如果不这样做将导致[DateTimeParseException](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/format/DateTimeParseException.html)：

```java
@Test(expected = DateTimeParseException.class)
public void shouldExpectAnExceptionIfTimeZoneIsMissing() {
    ZonedDateTime zonedDateTime = ZonedDateTime.parse("2021-11-01T05:06:20Z", DateTimeFormatter.ISO_INSTANT);
}
```

还值得一提的是，解析需要至少指定秒字段，否则，将抛出DateTimeParseException。

### 4.6 特定于语言环境的模式

Java 19引入了一个新方法ofLocalizedPattern(String)，其名称可能容易引起误解，因此值得在此提一下，该方法的目的并非像我们在前面示例中看到的ofPattern(String)与locale结合使用时那样提供本地化格式。

**此方法的主要目标是针对给定语言环境的模式提供最佳匹配**，其意图的最佳描述可以在[Unicode LDML规范](https://www.unicode.org/reports/tr35/tr35-dates.html#availableFormats_appendItems)中找到。他们以日本历法为例。在大多数情况下，年份应该与纪元相关联。在使用日语语言环境时，所有包含年份的模式都会与给定语言环境最合适的模式匹配，从而包含纪元。

**此方法的另一个不直观之处是，如果它无法匹配指定的模式，则会引发异常，这种情况可能经常发生**。即使模式有效并且可以使用ofP模式方法，代码也可能会出错。

## 5. 总结

在本文中，我们讨论了如何使用DateTimeFormatter类来格式化日期和时间。我们还研究了在处理日期时间实例时经常出现的实际示例模式。