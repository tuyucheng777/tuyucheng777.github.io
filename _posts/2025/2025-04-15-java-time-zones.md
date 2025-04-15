---
layout: post
title:  在Java中显示所有带有GMT和UTC的时区
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

每当我们处理时间和日期时，都需要一个参考框架。这个标准是[UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)，但在某些应用程序中我们也会见到[GMT](https://en.wikipedia.org/wiki/Greenwich_Mean_Time)。

简单来说，UTC是标准，GMT是时区。

关于如何使用，维基百科告诉我们如下内容：

> 在大多数情况下，UTC被认为可以与格林威治标准时间(GMT)互换，但科学界不再对GMT进行精确定义。

换句话说，一旦我们编制了包含UTC时区偏移量的列表，我们也会得到包含GMT时区偏移量的列表。

首先，我们来看看Java 8实现这一目标的方法，然后看看如何在Java 7中获得相同的结果。

## 2. 获取区域列表

首先，我们需要检索所有已定义时区的列表。

为此，ZoneId类有一个方便的静态方法：

```java
Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
```

然后，我们可以使用Set生成时区及其相应偏移量的排序列表：

```java
public List<String> getTimeZoneList(OffsetBase base) {
    LocalDateTime now = LocalDateTime.now();
    return ZoneId.getAvailableZoneIds().stream()
            .map(ZoneId::of)
            .sorted(new ZoneComparator())
            .map(id -> String.format(
                    "(%s%s) %s",
                    base, getOffset(now, id), id.getId()))
            .collect(Collectors.toList());
}
```

上述方法使用一个枚举参数来表示我们想要看到的偏移量：

```java
public enum OffsetBase {
    GMT, UTC
}
```

现在让我们更详细地看一下代码。

一旦我们检索到所有可用的区域ID，我们就需要一个实际的时间参考，由LocalDateTime.now()表示。

之后，我们使用Java的Stream API遍历时区字符串ID集合中的每个条目，并将其转换为具有相应偏移量的格式化时区列表。

对于每个条目，我们使用map(ZoneId::of)生成一个ZoneId实例。

## 3. 获取偏移量

我们还需要找到实际的UTC偏移量，例如，对于中欧时间，偏移量应为+01:00。

**要获取任何给定区域的UTC偏移量，我们可以使用LocalDateTime的getOffset()方法**。

还要注意，Java将+00:00偏移量表示为Z。

因此，为了使具有零偏移的时区的字符串看起来一致，我们将用+00:00替换Z：

```java
private String getOffset(LocalDateTime dateTime, ZoneId id) {
    return dateTime
            .atZone(id)
            .getOffset()
            .getId()
            .replace("Z", "+00:00");
}
```

## 4. 使区域Comparable

或者，我们也可以根据偏移量对时区进行排序。

为此，我们将使用ZoneComparator类：

```java
private class ZoneComparator implements Comparator<ZoneId> {

    @Override
    public int compare(ZoneId zoneId1, ZoneId zoneId2) {
        LocalDateTime now = LocalDateTime.now();
        ZoneOffset offset1 = now.atZone(zoneId1).getOffset();
        ZoneOffset offset2 = now.atZone(zoneId2).getOffset();

        return offset1.compareTo(offset2);
    }
}
```

## 5. 显示时区

剩下要做的就是通过为每个OffsetBase枚举值调用getTimeZoneList()方法并显示列表将上述部分放在一起：

```java
public class TimezoneDisplayApp {

    public static void main(String... args) {
        TimezoneDisplay display = new TimezoneDisplay();

        System.out.println("Time zones in UTC:");
        List<String> utc = display.getTimeZoneList(TimezoneDisplay.OffsetBase.UTC);
        utc.forEach(System.out::println);

        System.out.println("Time zones in GMT:");
        List<String> gmt = display.getTimeZoneList(TimezoneDisplay.OffsetBase.GMT);
        gmt.forEach(System.out::println);
    }
}
```

当我们运行上述代码时，它将打印UTC和GMT的时区。

以下是输出片段：

```text
Time zones in UTC:
(UTC+14:00) Pacific/Apia
(UTC+14:00) Pacific/Kiritimati
(UTC+14:00) Pacific/Tongatapu
(UTC+14:00) Etc/GMT-14
```

## 6. Java 7及之前版本

Java 8通过使用Stream和日期和时间API使这项任务变得更容易。

但是，如果我们有一个Java 7及之前版本的项目，我们仍然可以通过依赖java.util.TimeZone类及其getAvailableIDs()方法来实现相同的结果：

```java
public List<String> getTimeZoneList(OffsetBase base) {
    String[] availableZoneIds = TimeZone.getAvailableIDs();
    List<String> result = new ArrayList<>(availableZoneIds.length);

    for (String zoneId : availableZoneIds) {
        TimeZone curTimeZone = TimeZone.getTimeZone(zoneId);
        String offset = calculateOffset(curTimeZone.getRawOffset());
        result.add(String.format("(%s%s) %s", base, offset, zoneId));
    }
    Collections.sort(result);
    return result;
}
```

与Java 8代码的主要区别在于偏移量的计算。

**我们从TimeZone()的getRawOffset()方法中获得的rawOffset表示时区的偏移量(以毫秒为单位)**。

因此，我们需要使用TimeUnit类将其转换为小时和分钟：

```java
private String calculateOffset(int rawOffset) {
    if (rawOffset == 0) {
        return "+00:00";
    }
    long hours = TimeUnit.MILLISECONDS.toHours(rawOffset);
    long minutes = TimeUnit.MILLISECONDS.toMinutes(rawOffset);
    minutes = Math.abs(minutes - TimeUnit.HOURS.toMinutes(hours));

    return String.format("%+03d:%02d", hours, Math.abs(minutes));
}
```

## 7. 总结

在本快速教程中，我们了解了如何编制所有可用时区及其UTC和GMT偏移量的列表。