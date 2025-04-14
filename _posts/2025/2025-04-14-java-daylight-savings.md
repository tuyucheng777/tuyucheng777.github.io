---
layout: post
title:  在Java中处理夏令时
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

[夏令时](https://en.wikipedia.org/wiki/Daylight_saving_time)(或称DST)是一种在夏季提前时钟的做法，以便利用额外一小时的自然光(节省取暖电力、照明电力、改善心情等)。

它被[多个国家](https://en.wikipedia.org/wiki/Daylight_saving_time_by_country)使用，在处理日期和时间戳时需要考虑到它。

在本教程中，我们将了解如何根据不同位置正确在Java中处理DST。

## 2. JRE和DST可变性

首先，务必了解全球夏令时区域[经常变化](http://www.oracle.com/technetwork/java/javase/dst-faq-138158.html#change)，并且没有中央机构进行协调。

**一个国家，或者在某些情况下甚至是一个城市，可以决定是否以及如何应用或撤销它**。

每次发生这种情况时，更改都会记录在[IANA时区数据库](http://www.iana.org/time-zones)中，并且更新将在[JRE的未来版本](http://www.oracle.com/technetwork/java/javase/tzdata-versions-138805.html)中推出。

**如果无法等待，我们可以通过Oracle官方工具[Java时区更新工具(Java Time Zone Updater Tool)](http://www.oracle.com/technetwork/java/javase/documentation/timezones-137583.html)将包含新DST设置的修改后的时区数据强制放入JRE**，该工具可在[Java SE下载页面](http://www.oracle.com/technetwork/java/javase/downloads/index.html)上找到。

## 3. 错误方法：三个字母的时区ID

在JDK 1.1时代，API允许使用3个字母的时区ID，但这导致了一些问题。

首先，这是因为同一个3个字母的ID可能指代多个时区。例如，CST可能是美国的“中部标准时间”，也可能是“中国标准时间”。因此，Java平台只能识别其中一个。

另一个问题是，标准时区从不考虑夏令时，多个地区/区域/城市可以在同一个标准时区内使用各自的夏令时，因此标准时间不会遵循夏令时。

由于向后兼容，**仍然可以使用3个字母的ID实例化[java.util.Timezone](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TimeZone.html)。但是，此方法已弃用，不应再使用**。

## 4. 正确方法：TZDB时区ID

在Java中处理DST的正确方法是使用特定的TZDB时区ID实例化时区，例如“Europe/Rome”。

然后，我们将结合时间特定的类(如java.util.Calendar)使用它来获得TimeZone的原始偏移量(到GMT时区)的正确配置，以及自动DST偏移调整。

让我们看看当使用正确的时区时，如何自动处理从GMT+1到GMT+2的转变(发生在意大利，2018年3月25日，凌晨2:00)：

```java
TimeZone tz = TimeZone.getTimeZone("Europe/Rome");
TimeZone.setDefault(tz);
Calendar cal = Calendar.getInstance(tz, Locale.ITALIAN);
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm", Locale.ITALIAN);
Date dateBeforeDST = df.parse("2018-03-25 01:55");
cal.setTime(dateBeforeDST);
 
assertThat(cal.get(Calendar.ZONE_OFFSET)).isEqualTo(3600000);
assertThat(cal.get(Calendar.DST_OFFSET)).isEqualTo(0);
```

我们可以看到，ZONE_OFFSET为60分钟(因为意大利是GMT+1)，而当时DST_OFFSET为0。

让我们在Calendar中增加十分钟：

```java
cal.add(Calendar.MINUTE, 10);
```

现在DST_OFFSET也变成了60分钟，并且该国已将其当地时间从CET(中欧时间)转换为CEST(中欧夏令时)，即GMT+2：

```java
Date dateAfterDST = cal.getTime();
 
assertThat(cal.get(Calendar.DST_OFFSET))
    .isEqualTo(3600000);
assertThat(dateAfterDST)
    .isEqualTo(df.parse("2018-03-25 03:05"));
```

如果我们在控制台中显示这两个日期，我们还会看到时区也发生了变化：

```text
Before DST (00:55 UTC - 01:55 GMT+1) = Sun Mar 25 01:55:00 CET 2018
After DST (01:05 UTC - 03:05 GMT+2) = Sun Mar 25 03:05:00 CEST 2018
```

作为最后的测试，我们可以测量两个日期之间的距离，1:55和3:05：

```java
Long deltaBetweenDatesInMillis = dateAfterDST.getTime() - dateBeforeDST.getTime();
Long tenMinutesInMillis = (1000L * 60 * 10);
 
assertThat(deltaBetweenDatesInMillis)
    .isEqualTo(tenMinutesInMillis);
```

正如我们所料，距离是10分钟，而不是70分钟。

我们已经了解了如何通过正确使用TimeZone和Locale来避免在使用Date时可能遇到的常见陷阱。

## 5. 最佳方法：Java 8日期/时间API

使用这些线程不安全且并不总是用户友好的java.util类一直很困难，特别是由于兼容性问题导致它们无法被正确重构。

为此，Java 8引入了一个全新的包java.time，以及一套全新的API，即[Date/Time API](https://www.baeldung.com/java-8-date-time-intro)。它以ISO为中心，完全线程安全，并深受著名库Joda-Time的启发。

让我们仔细看看这个新类，从java.util.Date的后继者java.time.LocalDateTime开始：

```java
LocalDateTime localDateTimeBeforeDST = LocalDateTime
    .of(2018, 3, 25, 1, 55);
 
assertThat(localDateTimeBeforeDST.toString())
    .isEqualTo("2018-03-25T01:55");
```

我们可以观察LocalDateTime如何符合ISO8601配置文件，这是一种标准且广泛采用的日期时间符号。

但是，它完全不知道区域和偏移量，这就是为什么我们需要将其转换为**完全支持DST的java.time.ZonedDateTime**：

```java
ZoneId italianZoneId = ZoneId.of("Europe/Rome");
ZonedDateTime zonedDateTimeBeforeDST = localDateTimeBeforeDST
    .atZone(italianZoneId);
 
assertThat(zonedDateTimeBeforeDST.toString())
    .isEqualTo("2018-03-25T01:55+01:00[Europe/Rome]");
```

我们可以看到，现在日期包含两个基本尾随信息：+01:00是ZoneOffset，而[Europe/Rome\]是ZoneId。

与前面的示例类似，让我们通过增加十分钟来触发DST：

```java
ZonedDateTime zonedDateTimeAfterDST = zonedDateTimeBeforeDST
    .plus(10, ChronoUnit.MINUTES);
 
assertThat(zonedDateTimeAfterDST.toString())
    .isEqualTo("2018-03-25T03:05+02:00[Europe/Rome]");
```

再次，我们看到时间和区域偏移如何向前移动，并且仍然保持相同的距离：

```java
Long deltaBetweenDatesInMinutes = ChronoUnit.MINUTES
    .between(zonedDateTimeBeforeDST,zonedDateTimeAfterDST);
assertThat(deltaBetweenDatesInMinutes)
    .isEqualTo(10);
```

## 6. 总结

我们通过不同版本的Java核心API中的一些实际示例了解了夏令时是什么以及如何处理它。

在使用Java 8及更高版本时，鼓励使用新的java.time包，因为它易于使用且具有标准、线程安全的特性。