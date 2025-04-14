---
layout: post
title:  将Date转换为LocalDate或LocalDateTime并返回
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

从Java 8开始，我们有了一个新的Date API：java.time。

但是，有时我们仍然需要在新旧API之间进行转换，并使用两者的日期表示。

## 2. 将java.util.Date转换为java.time.LocalDate

在本节中，让我们看看将java.util.Date对象转换为新的java.time.LocalDate类型的方法。

### 2.1 使用toInstant()和toLocalDate() 

让我们首先将旧的日期表示形式转换为新的日期表示形式。

在这里，我们可以利用Java 8中添加到java.util.Date的新toInstant()方法。

**当我们转换Instant对象时，需要使用ZoneId，因为Instant对象与时区无关-只是时间线上的点**。

Instant的atZone(ZoneId zone) API返回一个ZonedDateTime，因此我们只需要使用toLocalDate()方法从中提取LocalDate。

首先，我们使用默认系统ZoneId：

```java
public LocalDate convertToLocalDateViaInstant(Date dateToConvert) {
    return dateToConvert.toInstant()
        .atZone(ZoneId.systemDefault())
        .toLocalDate();
}
```

请注意，对于1582年10月10日之前的日期，需要将Calendar设置为公历并调用方法[setGregorianChange](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/GregorianCalendar.html#setGregorianChange(java.util.Date)) ()：

```java
GregorianCalendar calendar = new GregorianCalendar();
calendar.setGregorianChange(new Date(Long.MIN_VALUE));
Date dateToConvert = calendar.getTime();
```

### 2.2 使用ofEpochMilli()和toLocalDate()

**类似的解决方案，但使用不同的方式创建Instant对象**-使用ofEpochMilli()方法：

```java
public LocalDate convertToLocalDateViaMilisecond(Date dateToConvert) {
    return Instant.ofEpochMilli(dateToConvert.getTime())
        .atZone(ZoneId.systemDefault())
        .toLocalDate();
}
```

其行为与之前的实现相同。

### 2.3 使用ofInstant()

在Java 9中，有一些新方法可以简化java.util.Date和java.time.LocalDate之间的转换。

让我们看一个例子：

```java
public LocalDate convertToLocalDate(Date dateToConvert) {
    return LocalDate.ofInstant(dateToConvert.toInstant(), ZoneId.systemDefault());
}
```

LocalDate.ofInstant(Instant instant, ZoneId zone)为转换提供了方便的快捷方式。

## 3. 将java.util.Date转换为java.time.LocalDateTime

现在让我们看看将java.util.Date转换为LocalDateTime实例的方法

### 3.1 使用toInstant()和toLocalDateTime()

要获取LocalDateTime实例，我们可以类似地**使用中间ZonedDateTime，然后使用toLocalDateTime() API**。

就像以前一样，我们可以使用两种可能的解决方案从java.util.Date获取Instant对象：

```java
public LocalDateTime convertToLocalDateTimeViaInstant(Date dateToConvert) {
    return dateToConvert.toInstant()
        .atZone(ZoneId.systemDefault())
        .toLocalDateTime();
}
```

### 3.2 使用ofEpochMilli()和toLocalDateTime()

除了toInstant()，我们还可以使用ofEpochMilli()：

```java
public LocalDateTime convertToLocalDateTimeViaMilisecond(Date dateToConvert) {
    return Instant.ofEpochMilli(dateToConvert.getTime())
        .atZone(ZoneId.systemDefault())
        .toLocalDateTime();
}
```

这将从Date实例返回LocalDateTime值。

### 3.3 使用ofInstant()

从Java 9开始，我们可以使用ofInstant()方法轻松地将java.util.Date转换为LocalDateTime：

```java
public LocalDateTime convertToLocalDateTime(Date dateToConvert) {
    return LocalDateTime.ofInstant(dateToConvert.toInstant(), ZoneId.systemDefault());
}
```

## 4. 将java.time转换为java.util.Date

现在我们已经很好地理解了如何从旧的数据表示形式转换为新的数据表示形式，让我们看看另一个方向的转换。

### 4.1 将java.time.LocalDate转换为java.util.Date

我们将讨论将LocalDate转换为Date的两种可能方法。

首先，我们**使用java.sql.Date对象提供的新的valueOf(LocalDate date)方法**，该方法以LocalDate作为参数：

```java
public Date convertToDateViaSqlDate(LocalDate dateToConvert) {
    return java.sql.Date.valueOf(dateToConvert);
}
```

我们看到，转换过程简单直观，它使用本地时区进行转换(所有操作都在后台完成，所以无需担心)。

在另一个Java 8示例中，我们使用一个Instant对象，将其传递给java.util.Date对象的from(Instant instant)方法：

```java
public Date convertToDateViaInstant(LocalDate dateToConvert) {
    return java.util.Date.from(dateToConvert.atStartOfDay()
        .atZone(ZoneId.systemDefault())
        .toInstant());
}
```

请注意，我们在这里使用了Instant对象，并且在进行此转换时我们还需要关注时区。

接下来，让我们使用非常相似的解决方案将LocalDateTime转换为Date对象。

### 4.2 将java.time.LocalDateTime转换为java.util.Date

**从LocalDateTime获取java.util.Date的最简单方法是使用java.sql.Timestamp的扩展**-可在Java 8中使用：

```java
public Date convertToDateViaSqlTimestamp(LocalDateTime dateToConvert) {
    return java.sql.Timestamp.valueOf(dateToConvert);
}
```

当然，另一种解决方案是**使用Instant对象，该对象可以从ZonedDateTime获得**：

```java
Date convertToDateViaInstant(LocalDateTime dateToConvert) {
    return java.util.Date
        .from(dateToConvert.atZone(ZoneId.systemDefault())
        .toInstant());
}
```

## 5. 将java.sql.Date转换为java.time.LocalDate

现在让我们快速看一下旧的java.sql.Date类以及如何将其转换为LocalDate。

从Java 8开始，我们可以在java.sql.Date上找到一个附加的toLocalDate()方法，它也为我们提供了一种将其转换为java.time.LocalDate的简单方法。

在这种情况下，我们不需要担心时区：

```java
public LocalDate convertToLocalDateViaSqlDate(Date dateToConvert) {
    return new java.sql.Date(dateToConvert.getTime()).toLocalDate();
}
```

从Java 8开始，我们还可以使用java.sql.Timestamp来获取LocalDateTime：

```java
public static LocalDateTime convertToLocalDateTimeViaSqlTimestamp(Date dateToConvert) {
    return new java.sql.Timestamp(dateToConvert.getTime()).toLocalDateTime();
}
```

## 6. 总结

在本文中，我们介绍了将旧java.util.Date转换为新java.time.LocalDate和java.time.LocalDateTime以及反之的各种方法，我们使用了toInstant()、ofEpochMilli()和ofInstant()等方法。此外，我们还探讨了将java.sql.Date和java.sql.Timestamp分别转换为LocalDate和LocalDateTime的方法。