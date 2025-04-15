---
layout: post
title:  在LocalDateTime和ZonedDateTime之间转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

Java LocalDateTime API表示并操作日期和时间的组合，ZonedDateTime是一个不可变对象，它包含一个精度为纳秒的日期时间值、一个基于[ISO 8601日历系统](https://en.wikipedia.org/wiki/ISO_8601)的时区值，以及一个用于处理模糊本地日期时间的[ZoneOffSet](https://www.baeldung.com/java-zone-offset)。

在本教程中，我们将了解如何从[LocalDateTime](https://www.baeldung.com/java-8-date-time-intro#3-working-with-localdatetime)转换为[ZonedDateTime](https://www.baeldung.com/java-8-date-time-intro#zonedDateTime)。

## 2. 将LocalDateTime转换为ZonedDateTime

让我们首先将LocalDateTime实例转换为ZonedDateTime。

### 2.1 使用atZone()方法

**LocalDateTime实例中的atZone()方法执行到ZonedDateTime的转换并维护相同的日期时间值**：

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 1, 1, 0, 30, 22);
ZonedDateTime zonedDateTime = localDateTime.atZone(ZoneId.of("Canada/Atlantic"));

assertEquals(localDateTime.getYear(), zonedDateTime.getYear());
assertEquals(localDateTime.getMonth(), zonedDateTime.getMonth());
assertEquals(localDateTime.getDayOfMonth(), zonedDateTime.getDayOfMonth());
assertEquals(localDateTime.getHour(), zonedDateTime.getHour());
assertEquals(localDateTime.getMinute(), zonedDateTime.getMinute());
assertEquals(localDateTime.getSecond(), zonedDateTime.getSecond());
```

atZone()方法接收基于ISO 8601日历系统指定时区的ZoneId值。

**调用withZoneSameInstant()方法使用ZoneOffSet时间差转换为实际的日期时间值**：

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 1, 1, 0, 30, 22);
ZonedDateTime zonedDateTime = localDateTime.atZone(ZoneId.of("Africa/Lagos")).withZoneSameInstant(ZoneId.of("Canada/Atlantic"));

assertEquals("2021-12-31T19:30:22-04:00[Canada/Atlantic]", zonedDateTime.toString());
assertEquals("-04:00", zonedDateTime.getOffset().toString());
```

我们可以通过调用静态ZoneId.getAvailableZoneIds()方法获取可用的ZoneId，此方法返回一组所有可用的基于区域的ID，以String的形式，我们可以从中选择并创建ZoneId对象。

此外，使用atZone()进行转换还附带一个ZoneOffSet值，该值提供ZonedDateTime对象和UTC(GMT)之间的时差(在上面的示例中为-04:00)。

### 2.2 使用ZonedDateTime.of()方法

ZonedDateTime类还提供了一个静态的of()方法来创建ZonedDateTime对象，该方法接收LocalDateTime和ZoneId的实例作为参数，并返回一个ZonedDateTime对象：

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 11, 5, 7, 30, 22);
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, ZoneId.of("Africa/Accra")).withZoneSameInstant(ZoneId.of("Africa/Lagos"));

assertEquals("2022-11-05T08:30:22+01:00[Africa/Lagos]", zonedDateTime.toString()); 
assertEquals(localDateTime.getYear(), zonedDateTime.getYear());
```

在这种情况下，正如我们之前看到的，我们可以通过调用withZoneSameInstant()方法来获取给定区域的实际日期时间值。

### 2.3 使用ZonedDateTime.ofInstant()方法

我们还可以将[ZoneOffSet](https://www.baeldung.com/java-zone-offset)对象与LocalDateTime结合使用来创建ZonedDateTime对象。

静态ofInstant()方法接收LocalDateTime、ZoneOffSet和ZoneId对象作为参数：

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 1, 5, 17, 30, 22);
ZoneId zoneId = ZoneId.of("Africa/Lagos");
ZoneOffset zoneOffset = zoneId.getRules().getOffset(localDateTime);
ZonedDateTime zonedDateTime = ZonedDateTime.ofInstant(localDateTime, zoneOffset, zoneId);

assertEquals("2022-01-05T17:30:22+01:00[Africa/Lagos]", zonedDateTime.toString());
```

**ZonedDateTime对象是通过组合LocalDateTime和ZoneOffSet对象隐式形成的Instant对象创建的**。

### 2.4 使用ZonedDateTime.ofLocal()方法

静态ofLocal()方法从LocalDateTime对象创建一个ZonedDateTime，并将首选的ZoneOffSet对象作为参数传递：

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 8 , 25, 8, 35, 22);
ZoneId zoneId = ZoneId.of("Africa/Lagos");
ZoneOffset zoneOffset = zoneId.getRules().getOffset(localDateTime);
ZonedDateTime zonedDateTime = ZonedDateTime.ofLocal(localDateTime, zoneId, zoneOffset);

assertEquals("2022-08-25T08:35:22+01:00[Africa/Lagos]", zonedDateTime.toString());
```

**通常，本地日期时间仅存在一个有效偏移量**。当发生时间重叠时，将有两个有效偏移量。

如果作为参数传递的首选ZoneOffset是有效偏移量之一，则使用该偏移量。否则，转换将保留先前的有效偏移量。

### 2.5 使用ZonedDateTime.ofStrict()方法

类似地，静态ofStrict()方法通过严格验证LocalDateTime、ZoneOffSet和ZoneID参数的组合来返回ZonedDateTime对象：

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 12, 25, 6, 18, 2);
ZoneId zoneId = ZoneId.of("Asia/Tokyo");
ZoneOffset zoneOffset = zoneId.getRules().getOffset(localDateTime);
ZonedDateTime zonedDateTime = ZonedDateTime.ofStrict(localDateTime, zoneOffset, zoneId);

assertEquals("2002-12-25T06:18:02+09:00[Asia/Tokyo]", zonedDateTime.toString());
```

如果我们提供的参数组合无效，该方法将抛出DateTimeException：

```java
zoneId = ZoneId.of("Asia/Tokyo");
zoneOffset = ZoneOffset.UTC;

assertThrows(DateTimeException.class, () -> ZonedDateTime.ofStrict(localDateTime, zoneOffset, zoneId));
```

上面的例子显示，当我们尝试使用来自Asia/Tokyo的ZoneId和代表默认UTC(GMT+0)的ZoneOffSet值的组合来创建ZonedDateTime对象时，会引发异常。

## 3. 将ZonedDateTime转换为LocalDateTime

ZonedDateTime对象维护三个不同的对象：LocalDateTime、ZoneId和ZoneOffset。

我们可以使用toLocalDateTime()方法将ZonedDateTime实例转换为LocalDateTime：

```java
ZonedDateTime zonedDateTime = ZonedDateTime.of(2011, 2, 12, 6, 14, 1, 58086000, ZoneId.of("Asia/Tokyo"));
LocalDateTime localDateTime = zonedDateTime.toLocalDateTime();

assertEquals("2011-02-12T06:14:01.058086+09:00[Asia/Tokyo]", zonedDateTime.toString());
```

此方法检索作为ZonedDateTime属性的一部分存储的LocalDateTime对象。

## 4. 总结

在本文中，我们学习了如何将LocalDateTime实例转换为ZonedDateTime。