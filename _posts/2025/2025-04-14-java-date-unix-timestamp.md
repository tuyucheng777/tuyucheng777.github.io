---
layout: post
title:  在Java中根据Unix时间戳创建日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在本快速教程中，我们将学习如何解析[Unix时间戳](https://www.baeldung.com/linux/date-command)中的日期表示。**[Unix时间](https://en.wikipedia.org/wiki/Unix_time)是自1970年1月1日以来经过的秒数，然而，时间戳可以表示精确到纳秒的时间**。因此，我们将了解可用的工具，并创建一个方法，将任意范围的时间戳转换为Java对象。

## 2. 旧方法(Java 8之前)

**在Java 8之前，最简单的选择是Date和Calendar**，Date类有一个构造函数，可以直接接收以毫秒为单位的时间戳：

```java
public static Date dateFrom(long input) {
    return new Date(input);
}
```

使用Calendar时，我们必须在getInstance()之后调用setTimeInMillis()：

```java
public static Calendar calendarFrom(long input) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTimeInMillis(input);
    return calendar;
}
```

换句话说，我们必须知道输入是以秒、纳秒还是任何其他精度为单位的。然后，我们必须手动将时间戳转换为毫秒。

## 3. 新方法(Java 8+)

Java 8引入了Instant，**该类提供了一些实用方法，可以根据秒和毫秒创建实例**。此外，其中一个方法还接收一个纳秒调整参数：

```java
Instant.ofEpochSecond(seconds, nanos);
```

**但我们仍然必须提前知道时间戳的精度**，例如，如果我们知道时间戳以纳秒为单位，则需要进行一些计算：

```java
public static Instant fromNanos(long input) {
    long seconds = input / 1_000_000_000;
    long nanos = input % 1_000_000_000;

    return Instant.ofEpochSecond(seconds, nanos);
}
```

首先，我们将时间戳除以十亿得到秒数。然后，用它的[余数](https://www.baeldung.com/modulo-java)得到秒数之后的部分。

## 4. Instant通用解决方案

**为了避免额外的工作，让我们创建一个方法，将任何输入转换为毫秒，大多数类都可**以解析。首先，我们检查时间戳的范围，然后，我们执行计算以提取毫秒数，此外，我们将使用科学计数法来使我们的条件更具可读性。

另外，请记住时间戳是[有符号](https://www.baeldung.com/java-unsigned-arithmetic)的，所以我们必须检查正值和负值范围(负时间戳意味着它们从1970年开始倒数)。

**那么，让我们首先检查我们的输入是否以纳秒为单位**：

```java
private static long millis(long timestamp) {
    if (millis >= 1E16 || millis <= -1E16) {
        return timestamp / 1_000_000;
    }

    // next range checks
}
```

首先，我们检查它是否在1E16范围内，即1后面跟着16个0。**负值表示1970年之前的日期，因此我们也需要检查它们**。然后，我们将该值除以一百万，得到精确到毫秒的数值。

同样，微秒在1E14范围内。这次，我们除以1000：

```java
if (timestamp >= 1E14 || timestamp <= -1E14) {
    return timestamp / 1_000;
}
```

**当我们的值在1E11到-3E10范围内时，我们不需要更改任何内**容，这意味着我们的输入已经是毫秒精度的了：

```java
if (timestamp >= 1E11 || timestamp <= -3E10) {
    return timestamp;
}
```

最后，如果我们的输入不是这些范围中的任何一个，那么它必须以秒为单位，所以我们需要将其转换为毫秒：

```java
return timestamp * 1_000;
```

### 4.1 标准化Instant输入

**现在，让我们创建一个方法，使用Instant.ofEpochMilli()从任意精度的输入中返回一个Instant**：

```java
public static Instant fromTimestamp(long input) {
    return Instant.ofEpochMilli(millis(input));
}
```

请注意，每次我们除或乘值时，精度都会丢失。

### 4.2 使用LocalDateTime获取本地时间

**Instant代表时间中的某个时刻，但是，如果没有[时区](https://www.baeldung.com/java-set-date-time-zone)，它就难以读取，因为它取决于我们在世界上的位置**。因此，让我们创建一个方法来生成本地时间表示，我们将使用UTC来避免测试中出现不同的结果：

```java
public static LocalDateTime localTimeUtc(Instant instant) {
    return LocalDateTime.ofInstant(instant, ZoneOffset.UTC);
}
```

**现在，我们可以测试一下，当方法需要特定格式时，使用错误的精度会导致完全不同的日期**。首先，我们传入一个以纳秒为单位的时间戳，我们已经知道正确的日期，但需要将其转换为微秒，并使用我们之前创建的fromNanos()方法：

```java
@Test
void givenWrongPrecision_whenInstantFromNanos_thenUnexpectedTime() {
    long microseconds = 1660663532747420283l / 1000;
    Instant instant = fromNanos(microseconds);
    String expectedTime = "2022-08-16T15:25:32";

    LocalDateTime time = localTimeUtc(instant);
    assertThat(!time.toString().startsWith(expectedTime));
    assertEquals("1970-01-20T05:17:43.532747420", time.toString());
}
```

当我们使用上一节中创建的fromTimestamp()方法时，不会发生此问题：

```java
@Test
void givenMicroseconds_whenInstantFromTimestamp_thenLocalTimeMatches() {
    long microseconds = 1660663532747420283l / 1000;

    Instant instant = fromTimestamp(microseconds);
    String expectedTime = "2022-08-16T15:25:32";

    LocalDateTime time = localTimeUtc(instant);
    assertThat(time.toString().startsWith(expectedTime));
}
```

## 5. 总结

在本文中，我们学习了如何使用核心Java类转换时间戳。然后，我们了解了时间戳如何具有不同的精度级别，以及这如何影响结果。最后，我们创建了一种简单的方法来规范化输入并获得一致的结果。