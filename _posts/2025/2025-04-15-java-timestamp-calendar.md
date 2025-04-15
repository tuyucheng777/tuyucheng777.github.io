---
layout: post
title:  将java.sql.Timestamp转换为java.util.Calendar
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在本教程中，我们将学习如何将[java.sql.Timestamp](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/security/Timestamp.html)对象转换为[java.util.Calendar](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Calendar.html)对象。

首先，我们将了解Java的两个类Timestamp和Calendar如何处理时间。然后，我们将探讨执行转换的常见用例。最后，我们将研究如何从一个对象转换为另一个对象。

## 2. Java中的时间

通常，我们处理的时间以毫秒为单位，一毫秒等于千分之一秒。如果需要更高的精度，另一个常用的精度是纳秒(十亿分之一秒)。

**java.sql.Timestamp类是java.util.Date对象的子类，以纳秒为单位的整数表示**。该类旨在与来自SQL数据库的时间戳数据类型一起使用，时间以自1970年1月1日00:00:00 GMT以来的毫秒数表示，并添加了前面提到的秒的小数部分以提高精度。

另一方面，**java.util.Calendar类允许我们从Timestamp中提取日期、月份或年份信息**。我们还可以使用它来获取未来的时间数据，或者扩展我们自己的Calendar实现。Calendar不支持高于毫秒的精度，我们稍后会探讨。

## 3. 转换时间对象

**转换这两种类型的主要用例是需要从数据库中提取时间戳字段**，我们的应用程序可能需要向用户显示一个值，或者在时间戳落在某个时间范围内时执行操作。通常，我们会尝试使用[Java 8](https://www.baeldung.com/java-8-date-time-intro)中引入的更新、功能更强大的java.time类，但由于一些遗留API，这并不总是可行的。

### 3.1 Timestamp转Calendar

无论哪种情况，一旦我们有了Timestamp对象，我们就可以将其转换为Calendar：

```java
Timestamp timestamp = new Timestamp(1713544200801L);
Calendar calendar = Calendar.getInstance();
calendar.setTimeInMillis(timestamp.getTime());
```

如上所示，**我们可以使用Timestamp的getTime()方法设置Calendar的时间**。为了简单起见，我们使用一个毫秒值的示例创建了一个Timestamp。为了确保转换正确，我们可以对其进行测试：

```java
assertEquals(calendar.getTimeInMillis(), timestamp.getTime());
```

一旦我们验证了转换正确，我们就可以根据需要继续使用Calendar。

### 3.2 Calendar转Timestamp

我们的测试确保两个对象的毫秒数匹配。然而，需要注意的是，**Timestamp的精度为纳秒，而当我们将Calendar转换回Timestamp时，就会失去该精度**：

```java
public static Timestamp calendarToTimestamp(Calendar calendar) {
    return new Timestamp(calendar.getTimeInMillis());
}
```

```java
int nanos = 801789562;
int losslessNanos = 801000000;
Timestamp timestamp = new Timestamp(1713544200801L);
timestamp.setNanos(nanos);
assertEquals(nanos, timestamp.getNanos());
Calendar calendar = SqlTimestampToCalendarConverter.timestampToCalendar(timestamp);
timestamp = SqlTimestampToCalendarConverter.calendarToTimestamp(calendar);
assertEquals(losslessNanos, timestamp.getNanos());
```

这里，当调用Timestamp的getTime()方法将其转换为毫秒时，其纳秒字段会使用整数除法内部除以1000000。这样，我们最终得到了从原始801789562纳秒值中减去801毫秒的结果。

因此，如果我们的任务需要纳秒精度，我们应该使用[另一种解决方案](https://www.baeldung.com/java-sql-timestamp-zoneddatetime-conversion)。

## 4. 总结

在本文中，我们了解到Timestamp和Calendar都以自纪元时间以来的毫秒数来处理时间。此外，我们发现Timestamp类可以跟踪纳秒，这对于我们的需求来说可能很有用。

通过探索这些对象的用法，我们学习了如何将Timestamp转换为Calendar，这在与特定日历字段(如日期或月份)交互时很有用。