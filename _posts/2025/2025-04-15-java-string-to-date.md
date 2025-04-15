---
layout: post
title:  在Java中将字符串转换为日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，**我们将探索将String对象转换为Date对象的几种方法**。我们将从Java 8中引入的新日期时间API java.time开始，然后再研究同样用于表示日期的旧java.util.Date数据类型。

最后，我们将研究一些使用Joda-Time和Apache Commons Lang DateUtils类进行转换的外部库。

## 2. 将String转换为LocalDate或LocalDateTime

[LocalDate](https://www.baeldung.com/java-8-date-time-intro)和LocalDateTime是不可变的日期时间对象，分别表示日期和日期时间。默认情况下，Java日期采用ISO-8601格式，因此，如果我们有任何表示此格式日期和时间的字符串，那么我们就**可以直接使用这些类的parse() API**。

### 2.1 使用Parse API

Date-Time API提供了parse()方法，用于解析包含日期和时间信息的字符串。**要将字符串对象转换为LocalDate和LocalDateTime对象，字符串必须符合[ISO_LOCAL_DATE](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE)或ISO_LOCAL_DATE_TIME格式，并表示有效的日期或时间**。

否则，运行时将抛出DateTimeParseException。

在我们的第一个例子中，让我们将String转换为java.time.LocalDate：

```java
LocalDate date = LocalDate.parse("2018-05-05");
```

可以使用与上述类似的方法将String转换为java.time.LocalDateTime：

```java
LocalDateTime dateTime = LocalDateTime.parse("2018-05-05T11:50:55");
```

需要注意的是，LocalDate和LocalDateTime对象都与时区无关。但是，**当我们需要处理特定于时区的日期和时间时，可以直接使用ZonedDateTime解析方法来获取特定于时区的日期时间**：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss z");
ZonedDateTime zonedDateTime = ZonedDateTime.parse("2015-05-05 10:15:30 Europe/Paris", formatter);
```

现在让我们看看如何使用自定义格式转换字符串。

### 2.2 使用带有自定义格式化器的Parse API

将自定义日期格式的字符串转换为Date对象是Java中普遍使用的操作。

为此，**我们将使用DateTimeFormatter类，它提供了许多预定义的格式化器，并允许我们定义格式化器**。

让我们从使用DateTimeFormatter的预定义格式化器之一的示例开始：

```java
String dateInString = "19590709";
LocalDate date = LocalDate.parse(dateInString, DateTimeFormatter.BASIC_ISO_DATE);
```

在下一个示例中，让我们创建一个应用“EEE,MMM d yyyy”格式的格式化器。此格式指定三个字符表示星期的完整名称，一个数字表示月份中的日期，三个字符表示月份，四位数字表示年份。

此格式化程序可识别诸如“Fri,3 Jan 2003”或“Wed,23 Mar 1994”之类的字符串：

```java
String dateInString = "Mon, 05 May 1980";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("EEE, d MMM yyyy", Locale.ENGLISH);
LocalDate dateTime = LocalDate.parse(dateInString, formatter);
```

### 2.3 常见的日期和时间模式

让我们看一些常见的日期和时间模式：

- y：年份(1996；96)
- M：月份(July；Jul；07)
- d：月份中的天数(1-31)
- E：星期几(星期五、星期日)
- a：AM/PM标记(AM、PM)
- H：一天中的小时数(0-23)
- h：AM/PM小时数(1-12)
- m：小时中的分钟数(0-60)
- s：分钟中的秒数(0-60)

要查看可用于指定解析模式的符号的完整列表，请单击[此处](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/format/DateTimeFormatter.html#patterns)。

如果我们需要将java.time日期转换为较旧的java.util.Date对象，请阅读[本文](https://www.baeldung.com/java-date-to-localdate-and-localdatetime)了解更多详细信息。

## 3. 将String转换为java.util.Date

**在Java 8之前，Java日期和时间机制是由java.util.Date、java.util.Calendar和java.util.TimeZone类的旧API提供的**，我们有时仍然需要使用这些API。

让我们看看如何将字符串转换为java.util.Date对象：

```java
SimpleDateFormat formatter = new SimpleDateFormat("dd-MMM-yyyy", Locale.ENGLISH);

String dateInString = "7-Jun-2013";
Date date = formatter.parse(dateInString);
```

在上面的例子中，我们首先需要通过传递描述日期和时间格式的模式来构造一个SimpleDateFormat对象。

接下来，我们需要调用parse()方法并传递日期字符串。如果传递的字符串参数与模式的格式不一致，则会抛出ParseException异常。

### 3.1 向java.util.Date添加时区信息 

**值得注意的是，java.util.Date没有时区概念**，只表示自Unix纪元时间-1970-01-01T00:00:00Z以来经过的秒数。

但是，当我们直接打印Date对象时，它将始终使用Java默认系统时区进行打印。

在最后一个例子中，我们将研究如何在添加时区信息的同时格式化日期：

```java
SimpleDateFormat formatter = new SimpleDateFormat("dd-M-yyyy hh:mm:ss a", Locale.ENGLISH);
formatter.setTimeZone(TimeZone.getTimeZone("America/New_York"));

String dateInString = "22-01-2015 10:15:55 AM"; 
Date date = formatter.parse(dateInString);
String formattedDateString = formatter.format(date);
```

我们也可以通过编程方式更改JVM时区，但不建议这样做：

```java
TimeZone.setDefault(TimeZone.getTimeZone("GMT"));
```

## 4. 外部库

现在我们已经很好地了解了如何使用核心Java提供的新旧API将String对象转换为Date对象，接下来让我们看一些外部库。

### 4.1 Joda-Time库

核心Java日期和时间库的替代方案是[Joda-Time](http://www.joda.org/joda-time/)，尽管作者现在建议用户迁移到java.time(JSR-310)，但如果无法迁移，**Joda-Time库提供了一个处理日期和时间的绝佳替代方案**。该库提供了Java 8日期时间项目中支持的几乎所有功能。

该依赖可以在[Maven Central](https://mvnrepository.com/search?q=joda-time)上找到：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

下面是使用标准DateTime的简单示例：

```java
DateTimeFormatter formatter = DateTimeFormat.forPattern("dd/MM/yyyy HH:mm:ss");

String dateInString = "07/06/2013 10:11:59";
DateTime dateTime = DateTime.parse(dateInString, formatter);
```

我们还来看一个明确设置时区的示例：

```java
DateTimeFormatter formatter = DateTimeFormat.forPattern("dd/MM/yyyy HH:mm:ss");

String dateInString = "07/06/2013 10:11:59";
DateTime dateTime = DateTime.parse(dateInString, formatter);
DateTime dateTimeWithZone = dateTime.withZone(DateTimeZone.forID("Asia/Kolkata"));
```

### 4.2 Apache Commons Lang-DateUtils

**DateUtils类提供了许多有用的实用程序，使得使用旧版Calendar和Date对象变得更加容易**。

commons-lang3依赖可从[Maven Central](https://mvnrepository.com/search?q=commons-lang3)获得：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

让我们使用日期模式数组将日期字符串转换为java.util.Date：

```java
String dateInString = "07/06-2013";
Date date = DateUtils.parseDate(dateInString, new String[] { "yyyy-MM-dd HH:mm:ss", "dd/MM-yyyy" });
```

## 5. 总结

在本文中，我们说明了将字符串转换为不同类型的Date对象(带时间或不带时间)的几种方法，包括使用纯Java以及使用外部库。