---
layout: post
title:  Java中12小时制到24小时制的转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在两种不同的时间格式之间进行转换是一项常见的编程任务，Java提供了用于时间操作的标准API。

在本教程中，我们将探讨如何使用[日期时间API](https://www.baeldung.com/java-8-date-time-intro)和旧版[日期API](https://www.baeldung.com/java-date-to-localdate-and-localdatetime)将12小时时间格式转换为24小时时间格式。

## 2. 使用日期时间API

**Java 8中引入的[Date Time API](https://www.baeldung.com/migrating-to-java-8-date-time-api)提供了一个类，用于使用不同的模式格式化时间**，12小时制和24小时制都有不同的表示模式。

下面是使用日期时间API将12小时制时间转换为24小时制时间的示例：

```java
@Test
void givenTimeInTwelveHours_whenConvertingToTwentyHoursWithDateTimeFormatter_thenConverted() throws ParseException {
    String time = LocalTime.parse("06:00 PM", DateTimeFormatter.ofPattern("hh:mm a", Locale.US))
        .format(DateTimeFormatter.ofPattern("HH:mm"));
    assertEquals("18:00", time);
}
```

在上面的代码中，我们通过调用LocalTime类上的parse()方法将12小时时间[字符串](https://www.baeldung.com/java-format-zoned-datetime-string)解析为LocalTime对象。

parse()方法接收两个参数-要解析的字符串和指定字符串格式的[DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter)。

接下来，我们通过调用DateTimeFormatter的ofPattern()方法将时间格式化为12小时制，**12小时制的模式为“hh:mm a”**。

此外，我们通过对解析的时间调用format()方法并将模式设置为“HH:mm”将12小时时间转换为24小时时间，这代表24小时时间格式。

## 3. 使用旧版日期API

简而言之，我们可以使用[SimpleDateFormat](https://www.baeldung.com/java-simple-date-format)和旧版Date API将12小时制时间转换为24小时制时间。

要使用旧版Date API，我们将把字符串解析为12小时时间格式的Date类型：

```java
@Test
public void givenTimeInTwelveHours_whenConvertingToTwentyHours_thenConverted() throws ParseException {
    SimpleDateFormat displayFormat = new SimpleDateFormat("HH:mm");
    SimpleDateFormat parseFormat = new SimpleDateFormat("hh:mm a");
    Date date = parseFormat.parse("06:00 PM");
    assertEquals("18:00", displayFormat.format(date));
}
```

这里，我们创建了一个SimpleDateFormat实例，通过指定模式将时间格式化为24小时制。最后，我们断言转换后的时间符合预期的24小时制。

## 4. 总结

在本文中，我们学习了两种将时间从12小时制转换为24小时制的方法。此外，我们还使用了Date Time API中的DateTimeFormatter类和旧版Date API中的SimpleDateFormat类来实现转换过程。