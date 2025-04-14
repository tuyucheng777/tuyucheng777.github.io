---
layout: post
title:  Java中自增日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将学习如何使用Java将日期增加一天。在Java 8之前，标准的Java日期和时间库并不十分友好。因此，Joda-Time成为了Java 8之前事实上的标准日期和时间库。

还有其他类和库可用于完成任务，如java.util.Calendar和Apache Commons。

Java 8包含更好的日期和时间API来解决其旧库的缺点。

因此，我们将研究**如何使用Java 8、Joda-Time API、Java的Calendar类和Apache Commons库将日期增加一天**。

## 2. Maven依赖

pom.xml文件中应包含以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>2.12.5</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.14.0</version>
    </dependency>
</dependencies>
```

你可以在[Maven Central](https://mvnrepository.com/artifact/joda-time/joda-time)上找到最新版本的Joda-Time，也可以找到最新版本的[Apache Commons Lang](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)。

## 3. 使用java.time

java.time.LocalDate类是不可变的日期时间表示，通常被视为年-月-日。

LocalDate有许多用于日期操作的方法，让我们看看如何使用它来完成相同的任务：

```java
public static String addOneDay(String date) {
    return LocalDate
        .parse(date)
        .plusDays(1)
        .toString();
}
```

在这个例子中，我们使用java.time.LocalDate类及其plusDays()方法将日期增加一天。

现在，让我们验证此方法是否按预期工作：

```java
@Test
public void givenDate_whenUsingJava8_thenAddOneDay() throws Exception {
    String incrementedDate = addOneDay("2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 4. 使用java.util.Calendar

另一种方法是使用java.util.Calendar及其add()方法来增加日期。

我们将使用它与java.text.SimpleDateFormat来实现日期格式化：

```java
public static String addOneDayCalendar(String date) throws ParseException {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    Calendar c = Calendar.getInstance();
    c.setTime(sdf.parse(date));
    c.add(Calendar.DATE, 1);
    return sdf.format(c.getTime());
}
```

java.text.SimpleDateFormat用于确保使用预期的日期格式，日期通过add()方法增加。

再次，让我们确保这种方法能够按预期发挥作用：

```java
@Test
public void givenDate_whenUsingCalendar_thenAddOneDay() throws Exception {
    String incrementedDate = addOneDayCalendar("2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 5. 使用Joda-Time

org.joda.time.DateTime类有许多方法可以帮助正确处理日期和时间。

让我们看看如何使用它将日期增加一天：

```java
public static String addOneDayJodaTime(String date) {
    DateTime dateTime = new DateTime(date);
    return dateTime
        .plusDays(1)
        .toString("yyyy-MM-dd");
}
```

在这里，我们使用org.joda.time.DateTime类及其plusDays()方法将日期增加一天。

我们可以通过以下单元测试来验证上述代码是否有效：

```java
@Test
public void givenDate_whenUsingJodaTime_thenAddOneDay() throws Exception {
    String incrementedDate = addOneDayJodaTime("2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 6. 使用Apache Commons

另一个常用于日期操作(以及其他用途)的库是Apache Commons，它是一套围绕java.util.Calendar和java.util.Date对象使用的实用程序。

对于我们的任务，我们可以使用org.apache.commons.lang3.time.DateUtils类及其addDays()方法(注意，SimpleDateFormat再次用于日期格式化)：

```java
public static String addOneDayApacheCommons(String date) throws ParseException {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    Date incrementedDate = DateUtils
        .addDays(sdf.parse(date), 1);
    return sdf.format(incrementedDate);
}
```

像往常一样，我们通过单元测试来验证结果：

```java
@Test
public void givenDate_whenUsingApacheCommons_thenAddOneDay() throws Exception {
    String incrementedDate = addOneDayApacheCommons("2018-07-03");
    assertEquals("2018-07-04", incrementedDate);
}
```

## 7. 总结

在这篇简短的文章中，我们探讨了处理日期加一天这一简单任务的各种方法，我们展示了如何使用Java核心API以及一些常用的第三方库来实现。