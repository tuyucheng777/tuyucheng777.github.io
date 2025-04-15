---
layout: post
title:  使用TimeUnit进行时间转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在Java中进行时间和持续时间计算时，TimeUnit枚举提供了一种在不同单位之间执行时间转换的便捷方法。

无论我们想将秒转换为分钟，将毫秒转换为小时，还是执行任何其他时间单位转换，我们都可以使用TimeUnit来简化代码，获得准确的结果，并使代码更具可读性。

在本教程中，我们将研究如何使用TimeUnit在Java中转换时间。

## 2. 理解TimeUnit

[TimeUnit](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html)是一个枚举类型，包含在java.util.concurrent包中，表示各种时间单位，范围从纳秒到天。它提供了一组预定义常量，每个常量对应一个特定的时间单位，包括：

- DAYS
- HOURS
- MINUTES
- SECONDS
- MILLISECONDS
- MICROSECONDS
- NANOSECONDS

这些常量是时间转换的基础。

## 3. 使用TimeUnit转换时间 

要执行时间转换，我们需要一个表示待转换时长的值，并指定要转换的目标单位。**TimeUnit提供了几种在单位之间转换时间的方法**，例如convert()或toXXX()，其中XXX代表目标单位。

### 3.1 使用convert()方法

首先，让我们看一下convert(long sourceDuration, TimeUnit sourceUnit)方法，它将给定单位的给定时间持续时间转换为枚举值指定的单位。

让我们将一个简单的整数转换为分钟：

```java
long minutes = TimeUnit.MINUTES.convert(60, TimeUnit.SECONDS);
assertThat(minutes).isEqualTo(1);
```

在此示例中，我们以60秒为起始值，然后将其转换为分钟，我们将源时间单位指定为第二个参数，输出单位始终由枚举值决定。

让我们看另一个例子，进行反向转换：

```java
long seconds = TimeUnit.SECONDS.convert(1, TimeUnit.MINUTES); 
assertThat(seconds).isEqualTo(60);
```

如我们所见，**convert()方法可以在不同的单位之间转换时间**。

### 3.2 使用toXXX()方法

现在让我们探索一下toXXX(long sourceDuration)方法，这里签名中的XXX指定了目标单位。

我们可以使用toNanos()、toMicros()、toMillis()、toSeconds()、toMinutes()、toHours()和toDays()选择单位。

现在让我们使用toXXX()方法重写前面的两个代码片段：

```java
long minutes = TimeUnit.SECONDS.toMinutes(60);
assertThat(minutes).isEqualTo(1);
```

和以前一样，我们只是将60秒转换为分钟。

我们也可以采用另一种方式进行转换：

```java
long seconds = TimeUnit.MINUTES.toSeconds(1);
assertThat(seconds).isEqualTo(60);
```

正如预期的那样，上述示例产生的结果与之前相同，因此两个签名是等效的。但与convert()不同，当我们使用toXXX()方法时，枚举值代表源时间单位。

## 4. 具体用例

现在我们知道了如何使用TimeUnit方法转换时间，让我们探索一些更详细的场景。

### 4.1 负输入

首先，让我们检查转换方法是否支持负输入值：

```java
long minutes = TimeUnit.SECONDS.toMinutes(-60);
assertThat(minutes).isEqualTo(-1);
```

给出的例子表明，**转换方法也可以处理负输入**，并且返回的结果仍然是正确的。

### 4.2 舍入处理

现在让我们检查一下，如果我们将一个较小的单位转换为一个较大的单位，并期望得到非整数结果，会发生什么。我们知道所有方法都只声明long作为返回类型，因此无法返回十进制结果。

让我们通过将秒转换为分钟来测试舍入规则：

```java
long positiveUnder = TimeUnit.SECONDS.toMinutes(59);
assertThat(positiveUnder).isEqualTo(0);
long positiveAbove = TimeUnit.SECONDS.toMinutes(61);
assertThat(positiveAbove).isEqualTo(1);
```

然后，让我们检查负面输入：

```java
long negativeUnder = TimeUnit.SECONDS.toMinutes(-59);
assertThat(negativeUnder).isEqualTo(0);

long negativeAbove = TimeUnit.SECONDS.toMinutes(-61);
assertThat(negativeAbove).isEqualTo(-1);
```

如我们所见，所有转换均已处理且没有任何错误。

我们应该注意，**所有方法都实现了向零舍入的规则，即截断小数部分**。

### 4.3 溢出处理

我们知道，所有基本类型都有其值的限制，不能超过。但是，如果结果超出了限制会发生什么？

让我们检查一下将天数的最小和最大长值转换为毫秒的结果：

```java
long maxMillis = TimeUnit.DAYS.toMillis(Long.MAX_VALUE);
assertThat(maxMillis).isEqualTo(Long.MAX_VALUE);
long minMillis = TimeUnit.DAYS.toMillis(Long.MIN_VALUE);
assertThat(minMillis).isEqualTo(Long.MIN_VALUE);
```

上述代码执行时没有抛出任何异常，**需要注意的是，结果无效**，因为溢出结果总是会被截断为long原语定义的最小值或最大值。

## 5. 转换为最精细单位

有时，我们可能需要将时长转换为TimeUnit中可用的最精细单位，例如将秒转换为小时、分钟和剩余秒数的适当组合。遗憾的是，目前没有方法可以实现这一点，**这是因为所有转换方法总是返回给定时长内完整周期的总数**。

**为了将输入转换为最精细的单位，我们需要实现一个自定义函数**，让我们使用3672秒的值并获取适当的时间单位组合-我们预期的值为1小时1分钟12秒。

为了提取小时数，我们可以使用：

```java
long inputSeconds = 3672;
long hours = TimeUnit.SECONDS.toHours(inputSeconds);
assertThat(hours).isEqualTo(1);
```

现在，如果我们想要提取剩余的分钟数，我们应该从输入值中减去小时数所包含的秒数之和，然后使用该值进行进一步的运算：

```java
long secondsRemainingAfterHours = inputSeconds - TimeUnit.HOURS.toSeconds(hours);
long minutes = TimeUnit.SECONDS.toMinutes(secondsRemainingAfterHours);
assertThat(minutes).isEqualTo(1);
```

我们刚刚根据输入的数据成功计算出了小时和分钟。最后，我们需要检索剩余的秒数。

为此，我们重复减法逻辑，并调整参数：

```java
long seconds = secondsRemainingAfterHours - TimeUnit.MINUTES.toSeconds(minutes);
assertThat(seconds).isEqualTo(12);
```

在示例中，我们将3672毫秒转换为最精细的单位表示，其中包括小时、分钟和秒。

## 6. 总结

在本文中，我们探讨了使用Java中的TimeUnit枚举转换时间的各种方法。

**TimeUnit枚举提供了一种使用convert()和toXXX()方法在不同单位之间进行转换的方便有效的方法**。

此外，它还能处理负输入并返回正确的结果。我们可以轻松地转换持续时间，无论是从较小的单位转换为较大的单位，还是从较大的单位转换为较小的单位，并且会向零舍入；它还通过将结果修剪为边界值来实现基本的溢出保护。

如果我们想将源时长转换为其他单位(例如天、小时、分钟和秒)的适当组合，可以轻松实现额外的逻辑，所有转换器都会返回指定时长内指定时间段的总数。