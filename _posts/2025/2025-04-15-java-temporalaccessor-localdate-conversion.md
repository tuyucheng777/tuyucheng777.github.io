---
layout: post
title:  将TemporalAccessor转换为LocalDate
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

处理[日期和时间](https://www.baeldung.com/java-8-date-time-intro)值是一项常见的任务，有时，我们可能需要将TemporalAccessor对象转换为LocalDate对象来执行特定于日期的操作。因此，这在解析[日期时间字符串](https://www.baeldung.com/java-string-to-date)或从日期时间对象中提取日期部分时非常有用。

**在本教程中，我们将探索在Java中实现这种转换的不同方法**。

## 2. 使用LocalDate.from()方法

将[TemporalAccessor](https://www.baeldung.com/java-8-localization#1-datetimeformatter)转换为[LocalDate](https://www.baeldung.com/java-date-to-localdate-and-localdatetime)的一个直接方法是使用LocalDate.from(TemporalAccessor temporary)方法。实际上，此方法从TemporalAccessor中提取日期组件(年、月、日)并构造一个LocalDate对象，我们来看一个例子：

```java
String dateString = "2022-03-28";
TemporalAccessor temporalAccessor = DateTimeFormatter.ISO_LOCAL_DATE.parse(dateString);
```

```java
@Test
public void givenTemporalAccessor_whenUsingLocalDateFrom_thenConvertToLocalDate() {
    LocalDate convertedDate = LocalDate.from(temporalAccessor);
    assertEquals(LocalDate.of(2022, 3, 28), convertedDate);
}
```

在此代码片段中，我们初始化一个字符串变量dateString，其值为(2022-03-28)，表示ISO 8601格式的日期。此外，我们使用DateTimeFormatter.ISO_LOCAL_DATE.parse()方法将此字符串解析为TemporalAccessor对象temporaryAccessor。

**随后，我们使用LocalDate.from(temporalAccessor)方法将temporaryAccessor转换为LocalDate对象convertDate，从而有效地提取和构造日期组件**。

最后，通过断言assertEquals(LocalDate.of(2022,3,28), convertedDate)，我们确保转换结果convertedDate与预期日期匹配。

## 3. 使用TemporalQueries

将TemporalAccessor转换为LocalDate的另一种方法是使用TemporalQueries，我们可以定义一个自定义的[TemporalQuery](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/temporal/TemporalQuery.html)来提取必要的日期组件并构造一个LocalDate对象；以下是示例：

```java
@Test
public void givenTemporalAccessor_whenUsingTemporalQueries_thenConvertToLocalDate() {
    int year = temporalAccessor.query(TemporalQueries.localDate()).getYear();
    int month = temporalAccessor.query(TemporalQueries.localDate()).getMonthValue();
    int day = temporalAccessor.query(TemporalQueries.localDate()).getDayOfMonth();

    LocalDate convertedDate = LocalDate.of(year, month, day);
    assertEquals(LocalDate.of(2022, 3, 28), convertedDate);
}
```

在这个测试方法中，我们调用temporaryAccessor.query(TemporalQueries.localDate())方法来获取一个表示从temporaryAccessor中提取的日期的LocalDate实例。

然后，我们分别使用getYear()、getMonthValue()和getDayOfMonth()方法从此LocalDate实例中检索年、月、日组件。随后，我们使用LocalDate.of()方法，根据这些提取的组件构造一个LocalDate对象convertDate。

## 4. 总结

总而言之，在Java中，可以使用LocalDate.from()或TemporalQueries将TemporalAccessor转换为LocalDate。此外，这些方法提供了灵活高效的转换方式，从而能够在Java应用程序中无缝集成日期时间功能。