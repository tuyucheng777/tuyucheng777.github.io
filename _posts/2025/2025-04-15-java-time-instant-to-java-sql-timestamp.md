---
layout: post
title:  在java.time.Instant和java.sql.Timestamp之间转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

java.time.Instant和java.sql.Timestamp类都表示UTC时间轴上的一个点，换句话说，它们表示自[Java纪元](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/Instant.html)以来的纳秒数。

在本快速教程中，我们将使用内置Java方法介绍它们之间的转换。

## 2. Instant与Timestamp之间的相互转换

**我们可以使用Timestamp.from()将Instant转换为Timestamp**：

```java
Instant instant = Instant.now();
Timestamp timestamp = Timestamp.from(instant);
assertEquals(instant.toEpochMilli(), timestamp.getTime());
```

反之亦然，我们可以使用Timestamp.toInstant()将Timestamp转换为Instant：

```java
instant = timestamp.toInstant();
assertEquals(instant.toEpochMilli(), timestamp.getTime());
```

**无论哪种方式，Instant和Timestamp都代表时间线上的同一点**。

接下来我们看一下这两个类与时区之间的交互。

## 3. toString()方法差异

**在Instant和Timestamp上调用toString()时，其行为会因时区而异**。Instant.toString()返回UTC时区的时间，而Timezone.toString()返回本地机器时区的时间。

让我们看看分别在instant和timestamp上调用toString()时会得到什么：

```text
Instant (in UTC): 2018-10-18T00:00:57.907Z
Timestamp (in GMT +05:30): 2018-10-18 05:30:57.907
```

这里，timestamp.toString()返回的时间比instant.toString()返回的时间晚了5小时30分钟，这是因为本地机器的时区是GMT+5:30时区。

**toString()方法的输出不同，但Timestamp和Instant都代表时间线上的同一点**。

我们还可以通过将Timestamp转换为UTC时区来验证这一点：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");
formatter = formatter.withZone(TimeZone.getTimeZone("UTC").toZoneId());
DateFormat df = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");

assertThat(formatter.format(instant)).isEqualTo(df.format(timestamp));
```

## 4. 总结

在本快速教程中，我们了解了如何使用内置方法在Java中的java.time.Instant和java.sql.Timestamp类之间进行转换。

我们还研究了时区如何影响输出的变化。