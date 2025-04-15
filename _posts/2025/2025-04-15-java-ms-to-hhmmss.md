---
layout: post
title:  将毫秒持续时间格式化为HH:MM:SS
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

持续时间是以小时、分钟、秒、毫秒等单位表示的时间量，我们可能希望将持续时间格式化为某种特定的时间模式。

我们可以通过借助一些JDK库编写自定义代码或利用第三方库来实现这一点。

在本快速教程中，我们将了解如何编写简单的代码将给定的持续时间格式化为HH:MM:SS格式。

## 2. Java解决方案

有多种方式可以表示持续时间-例如，以分钟、秒和毫秒为单位，或者以Java Duration为单位，它有自己的特定格式。

本节及后续章节将重点介绍如何使用一些JDK库将以毫秒为单位的时间间隔(已用时间)格式化为HH:MM:SS。为了便于示例，我们将38114000ms格式化为10:35:14(HH:MM:SS)。

### 2.1 Duration

**从Java 8开始，引入了[Duration](https://www.baeldung.com/java-period-duration)类来处理各种单位的时间间隔**。Duration类提供了许多辅助方法，可以从持续时间中获取小时、分钟和秒。

要使用Duration类将间隔格式化为HH:MM:SS，我们需要使用Duration类中的工厂方法ofMillis初始化间隔对应的Duration对象，这会将间隔转换为我们可以处理的Duration对象：

```java
Duration duration = Duration.ofMillis(38114000);
```

为了便于从秒数计算到我们想要的单位，我们需要获取持续时间或间隔内的总秒数：

```java
long seconds = duration.getSeconds();
```

然后，一旦我们有了秒数，我们就会根据所需的格式生成相应的小时、分钟和秒：

```java
long HH = seconds / 3600;
long MM = (seconds % 3600) / 60;
long SS = seconds % 60;
```

最后，我们格式化生成的值：

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
```

让我们尝试一下这个解决方案：

```java
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

**如果我们使用的是Java 9或更高版本，我们可以使用一些辅助方法直接获取单位，而无需执行任何计算**：

```java
long HH = duration.toHours();
long MM = duration.toMinutesPart();
long SS = duration.toSecondsPart();
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
```

上面的代码片段将给出与上面测试相同的结果：

```java
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

### 2.2 TimeUnit

就像上一节讨论的Duration类一样，TimeUnit表示给定粒度的时间。它提供了一些辅助方法来跨单位转换(在我们的例子中是小时、分钟和秒)，并以这些单位执行计时和延迟操作。

要将毫秒持续时间格式化为HH:MM:SS格式，我们需要做的就是使用TimeUnit中相应的辅助方法：

```java
long HH = TimeUnit.MILLISECONDS.toHours(38114000);
long MM = TimeUnit.MILLISECONDS.toMinutes(38114000) % 60;
long SS = TimeUnit.MILLISECONDS.toSeconds(38114000) % 60;
```

然后，根据上面生成的单位格式化持续时间：

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

## 3. 使用第三方库

我们可以选择尝试使用第三方库方法而不是自己编写实现。

### 3.1 Apache Commons

要使用[Apache Commons](https://www.baeldung.com/java-commons-lang-3)，我们需要将[commons-lang3](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)添加到项目中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

正如预期的那样，该库在其DurationFormatUtils类中具有formatDuration以及其他单位格式化方法：

```java
String timeInHHMMSS = DurationFormatUtils.formatDuration(38114000, "HH:MM:SS", true);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

### 3.2 Joda Time

**当我们使用Java 8之前的Java版本时，[Joda Time](https://www.baeldung.com/joda-time)库会非常方便**，因为它提供了一些方便的辅助方法来表示和格式化时间单位。要使用Joda Time，我们需要在项目中添加[joda-time依赖](https://mvnrepository.com/artifact/joda-time/joda-time)：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10.10</version>
</dependency>
```

Joda Time有一个Duration类来表示时间，首先，我们将时间间隔(以毫秒为单位)转换为Joda Time Duration对象的实例：

```java
Duration duration = new Duration(38114000);
```

然后，我们使用Duration中的toPeriod方法从上面的持续时间中获取周期，该方法将其转换或初始化为Joda Time中Period类的实例：

```java
Period period = duration.toPeriod();
```

我们使用相应的辅助方法从Period获取单位(小时、分钟和秒)：

```java
long HH = period.getHours();
long MM = period.getMinutes();
long SS = period.getSeconds();
```

最后，我们可以格式化持续时间并测试结果：

```java
String timeInHHMMSS = String.format("%02d:%02d:%02d", HH, MM, SS);
assertThat(timeInHHMMSS).isEqualTo("10:35:14");
```

## 4. 总结

在本教程中，我们学习了如何将持续时间格式化为特定格式(在我们的例子中为HH:MM:SS)。

首先，我们使用Java自带的Duration和TimeUnit类来获取所需的单位，并借助Formatter对其进行格式化。

最后，我们研究了如何使用一些第三方库来实现结果。