---
layout: post
title:  如何在Java中计算“过去了多久”
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

计算相对时间和两个时间点之间的持续时间是软件系统中的常见用例；例如，我们可能想向用户显示自某个事件(例如在社交媒体平台上发布一张新照片)以来已经过去了多长时间。这类“过去一段时间”文本的示例包括“5分钟前”、“1年前”等。

虽然语义和词语的选择完全取决于上下文，但总体思想是相同的。

在本教程中，我们将探索Java中计算时间之前的几种解决方案。**因为[Java 8中引入了新的日期和时间API](https://www.baeldung.com/java-8-date-time-intro)，我们将分别讨论Java 7和Java 8版本的解决方案**。

## 2. Java 7版本

Java 7中有几个与时间相关的类。此外，由于Java 7 Date API的缺陷，还提供了一些第三方时间和日期库。

首先，让我们使用纯Java 7来计算“过去了多久”。

### 2.1 纯Java 7

我们定义一个枚举，它包含不同的时间粒度并将它们转换为毫秒：

```java
public enum TimeGranularity {
    SECONDS {
        public long toMillis() {
            return TimeUnit.SECONDS.toMillis(1);
        }
    }, MINUTES {
        public long toMillis() {
            return TimeUnit.MINUTES.toMillis(1);
        }
    }, HOURS {
        public long toMillis() {
            return TimeUnit.HOURS.toMillis(1);
        }
    }, DAYS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(1);
        }
    }, WEEKS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(7);
        }
    }, MONTHS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(30);
        }
    }, YEARS {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(365);
        }
    }, DECADES {
        public long toMillis() {
            return TimeUnit.DAYS.toMillis(365 * 10);
        }
    };

    public abstract long toMillis();
}
```

我们使用了java.util.concurrent.TimeUnit枚举，它是一个强大的时间转换工具。使用TimeUnit枚举，我们为TimeGranularity枚举的每个值重写了toMillis()抽象方法，以便它返回与每个值对应的毫秒数。例如，对于“十年”，它返回3650天的毫秒数。

定义TimeGranularity枚举后，我们可以定义两个方法，第一个方法接收一个java.util.Date对象和一个TimeGranularity实例，并返回一个“time ago”字符串：

```java
static String calculateTimeAgoByTimeGranularity(Date pastTime, TimeGranularity granularity) {
    long timeDifferenceInMillis = getCurrentTime() - pastTime.getTime();
    return timeDifferenceInMillis / granularity.toMillis() + " " + granularity.name().toLowerCase() + " ago";
}
```

此方法将当前时间与给定时间的差值除以TimeGranularity值(以毫秒为单位)，这样，我们就可以粗略地计算出在指定的时间粒度内，自给定时间以来经过的时间量。

我们使用了getCurrentTime()方法获取当前时间，为了进行测试，我们返回一个固定的时间点，避免从本地机器读取时间。实际应用中，此方法会使用System.currentTimeMillis()或LocalDateTime.now()返回当前时间的实际值。

让我们测试一下这个方法：

```java
Assert.assertEquals("5 hours ago",
                    TimeAgoCalculator.calculateTimeAgoByTimeGranularity(new Date(getCurrentTime() - (5 * 60 * 60 * 1000)), TimeGranularity.HOURS));
```

此外，我们还可以编写一种方法，自动检测最大的合适时间粒度并返回更人性化的输出：

```java
static String calculateHumanFriendlyTimeAgo(Date pastTime) {
    long timeDifferenceInMillis = getCurrentTime() - pastTime.getTime();
    if (timeDifferenceInMillis / TimeGranularity.DECADES.toMillis() > 0) {
        return "several decades ago";
    } else if (timeDifferenceInMillis / TimeGranularity.YEARS.toMillis() > 0) {
        return "several years ago";
    } else if (timeDifferenceInMillis / TimeGranularity.MONTHS.toMillis() > 0) {
        return "several months ago";
    } else if (timeDifferenceInMillis / TimeGranularity.WEEKS.toMillis() > 0) {
        return "several weeks ago";
    } else if (timeDifferenceInMillis / TimeGranularity.DAYS.toMillis() > 0) {
        return "several days ago";
    } else if (timeDifferenceInMillis / TimeGranularity.HOURS.toMillis() > 0) {
        return "several hours ago";
    } else if (timeDifferenceInMillis / TimeGranularity.MINUTES.toMillis() > 0) {
        return "several minutes ago";
    } else {
        return "moments ago";
    }
}
```

现在，让我们看一个测试来了解示例用法：

```java
Assert.assertEquals("several hours ago", TimeAgoCalculator.calculateHumanFriendlyTimeAgo(new Date(getCurrentTime() - (5 * 60 * 60 * 1000))));
```

根据上下文，我们可以使用不同的词语，如“few”，“many”，“很多”，甚至是确切的值。

### 2.2 Joda-Time库

在Java 8发布之前，[Joda-Time](https://www.baeldung.com/joda-time)是Java中各种时间和日期相关操作的事实标准，我们可以使用Joda-Time库中的三个类来计算“过去了多久”：

- org.joda.time.Period接收两个org.joda.time.DateTime对象并计算这两个时间点之间的差值
- org.joda.time.format.PeriodFormatter定义打印Period对象的格式
- org.joda.time.format.PeriodFormatBuilder是一个用于创建自定义PeriodFormatter的构建器类

我们可以使用这三个类轻松获取现在和过去某个时间之间的准确时间：

```java
static String calculateExactTimeAgoWithJodaTime(Date pastTime) {
    Period period = new Period(new DateTime(pastTime.getTime()), new DateTime(getCurrentTime()));
    PeriodFormatter formatter = new PeriodFormatterBuilder().appendYears()
            .appendSuffix(" year ", " years ")
            .appendSeparator("and ")
            .appendMonths()
            .appendSuffix(" month ", " months ")
            .appendSeparator("and ")
            .appendWeeks()
            .appendSuffix(" week ", " weeks ")
            .appendSeparator("and ")
            .appendDays()
            .appendSuffix(" day ", " days ")
            .appendSeparator("and ")
            .appendHours()
            .appendSuffix(" hour ", " hours ")
            .appendSeparator("and ")
            .appendMinutes()
            .appendSuffix(" minute ", " minutes ")
            .appendSeparator("and ")
            .appendSeconds()
            .appendSuffix(" second", " seconds")
            .toFormatter();
    return formatter.print(period);
}
```

让我们看一个示例用法：

```java
Assert.assertEquals("5 hours and 1 minute and 1 second", 
    TimeAgoCalculator.calculateExactTimeAgoWithJodaTime(new Date(getCurrentTime() - (5 * 60 * 60 * 1000 + 1 * 60 * 1000 + 1 * 1000))));
```

还可以生成更加人性化的输出：

```java
static String calculateHumanFriendlyTimeAgoWithJodaTime(Date pastTime) {
    Period period = new Period(new DateTime(pastTime.getTime()), new DateTime(getCurrentTime()));
    if (period.getYears() != 0) {
        return "several years ago";
    } else if (period.getMonths() != 0) {
        return "several months ago";
    } else if (period.getWeeks() != 0) {
        return "several weeks ago";
    } else if (period.getDays() != 0) {
        return "several days ago";
    } else if (period.getHours() != 0) {
        return "several hours ago";
    } else if (period.getMinutes() != 0) {
        return "several minutes ago";
    } else {
        return "moments ago";
    }
}
```

我们可以运行一个测试来查看该方法是否返回更加人性化的“time ago”字符串：

```java
Assert.assertEquals("several hours ago", 
    TimeAgoCalculator.calculateHumanFriendlyTimeAgoWithJodaTime(new Date(getCurrentTime() - (5 * 60 * 60 * 1000))));
```

同样，根据用例，我们可以使用不同的术语，例如“one”、“few”或“many”。

### 2.3 Joda-Time时区

使用Joda-Time库在“过去了多久”的计算中添加时区非常简单：

```java
String calculateZonedTimeAgoWithJodaTime(Date pastTime, TimeZone zone) {
    DateTimeZone dateTimeZone = DateTimeZone.forID(zone.getID());
    Period period = new Period(new DateTime(pastTime.getTime(), dateTimeZone), new DateTime(getCurrentTimeByTimeZone(zone)));
    return PeriodFormat.getDefault().print(period);
}
```

getCurrentTimeByTimeZone()方法返回指定时区中的当前时间值。出于测试目的，此方法会返回一个固定的时间点，但实际应用中，应该使用Calendar.getInstance(zone).getTimeInMillis()或LocalDateTime.now(zone)返回当前时间的实际值。

## 3. Java 8

[Java 8引入了全新改进的日期和时间API](https://www.baeldung.com/java-8-date-time-intro)，它借鉴了Joda-Time库的许多思想，**我们可以使用原生的java.time.Duration和java.time.Period类来计算“过去了多久”**：

```java
static String calculateTimeAgoWithPeriodAndDuration(LocalDateTime pastTime, ZoneId zone) {
    Period period = Period.between(pastTime.toLocalDate(), getCurrentTimeByTimeZone(zone).toLocalDate());
    Duration duration = Duration.between(pastTime, getCurrentTimeByTimeZone(zone));
    if (period.getYears() != 0) {
        return "several years ago";
    } else if (period.getMonths() != 0) {
        return "several months ago";
    } else if (period.getDays() != 0) {
        return "several days ago";
    } else if (duration.toHours() != 0) {
        return "several hours ago";
    } else if (duration.toMinutes() != 0) {
        return "several minutes ago";
    } else if (duration.getSeconds() != 0) {
        return "several seconds ago";
    } else {
        return "moments ago";
    }
}
```

上述代码片段支持时区，并且仅使用原生Java 8 API。

## 4. PrettyTime库

**PrettyTime是一个功能强大的库，它专门提供“过去了多久”功能并支持i18n**。此外，它高度可定制，易于使用，并且可以与Java 7和8版本一起使用。

首先，让我们将它的[依赖](https://mvnrepository.com/artifact/org.ocpsoft.prettytime/prettytime/3.2.7.Final)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.ocpsoft.prettytime</groupId>
    <artifactId>prettytime</artifactId>
    <version>3.2.7.Final</version>
</dependency>
```

现在，以人性化的格式获取“过去了多久”非常容易：

```java
String calculateTimeAgoWithPrettyTime(Date pastTime) {
    PrettyTime prettyTime = new PrettyTime();
    return prettyTime.format(pastTime);
}
```

## 5. Time4J库

最后，Time4J是另一个用于处理Java中时间和日期数据的优秀库，它有一个PrettyTime类，可以用来计算过去的时间。

让我们添加它的[依赖](https://mvnrepository.com/artifact/net.time4j)：

```xml
<dependency>
    <groupId>net.time4j</groupId>
    <artifactId>time4j-base</artifactId>
    <version>5.9</version>
</dependency>
<dependency>
    <groupId>net.time4j</groupId>
    <artifactId>time4j-sqlxml</artifactId>
    <version>5.8</version>
</dependency>
```

添加此依赖后，计算时间就非常简单了：

```java
String calculateTimeAgoWithTime4J(Date pastTime, ZoneId zone, Locale locale) {
    return PrettyTime.of(locale).printRelative(pastTime.toInstant(), zone);
}
```

与PrettyTime库相同，Time4J也开箱即用地支持i18n。

## 6. 总结

在本文中，我们讨论了Java中计算时间前的不同方法。

Java 8引入了新的日期和时间API，因此Java 8之前和之后版本的纯Java解决方案有所不同。