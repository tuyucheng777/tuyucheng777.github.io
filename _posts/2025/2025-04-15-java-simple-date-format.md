---
layout: post
title:  SimpleDateFormat指南
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在本教程中，我们将深入了解SimpleDateFormat类。

我们将了解简单的**实例和格式化样式**以及该类用于处理语言环境和时区的有用方法。

## 2. 简单实例

首先，让我们看看如何实例化一个新的SimpleDateFormat对象。

有[4个可能的构造函数](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/text/SimpleDateFormat.html#constructor-summary)-但为了与名称保持一致，我们尽量简化，**我们只需要一个表示所需[日期模式](https://www.baeldung.com/java-simple-date-format#date_time_patterns)的字符串即可**。

让我们从破折号分隔的日期模式开始，如下所示：

```text
"dd-MM-yyyy"
```

这将正确地格式化日期，从当前日期开始，到当前月份，最后到当前年份。我们可以用一个简单的单元测试来测试新的格式化程序，我们将实例化一个新的SimpleDateFormat对象，并传入一个已知日期：

```java
SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy");
assertEquals("24-05-1977", formatter.format(new Date(233345223232L)));
```

在上面的代码中，格式化程序将毫秒转换为人类可读的日期-1977年5月24日。

### 2.1 工厂方法

尽管SimpleDateFormat是一个可以快速构建日期格式化程序的便捷类，但我们还是**鼓励使用DateFormat类上的工厂方法getDateFormat()、getDateTimeFormat()、getTimeFormat()**。

当使用这些工厂方法时，上面的例子看起来有点不同：

```java
DateFormat formatter = DateFormat.getDateInstance(DateFormat.SHORT);
assertEquals("5/24/77", formatter.format(new Date(233345223232L)));
```

从上面可以看出，格式化选项的数量是由[DateFormat类中的字段](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/text/DateFormat.html#field-summary)预先确定的，这在很大程度上限制了我们可用的格式化选项，这就是为什么我们在本文中将坚持使用SimpleDateFormat的原因。

### 2.2 线程安全

SimpleDateFormat的[JavaDoc](https://github.com/openjdk/jdk/blob/76507eef639c41bffe9a4bb2b8a5083291f41383/src/java.base/share/classes/java/text/SimpleDateFormat.java#L427)明确指出：

> 日期格式不同步，建议为每个线程创建单独的格式实例。如果多个线程并发访问某个格式，则必须在外部进行同步。

**因此SimpleDateFormat实例不是线程安全的**，我们应该在并发环境中谨慎使用它们。

解决这个问题的最佳方法是将它们与[ThreadLocal](https://www.baeldung.com/java-threadlocal) 结合使用，这样，每个线程最终都会拥有自己的SimpleDateFormat实例，并且由于不共享，程序是线程安全的： 

```java
private final ThreadLocal<SimpleDateFormat> formatter = ThreadLocal
    .withInitial(() -> new SimpleDateFormat("dd-MM-yyyy"));
```

[withInitial](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ThreadLocal.html#withInitial(java.util.function.Supplier))方法的参数是SimpleDateFormat实例的Supplier，每次ThreadLocal需要创建实例时，都会使用这个Supplier。

然后我们可以通过ThreadLocal实例使用格式化程序：

```java
formatter.get().format(date)
```

ThreadLocal.get()方法首先为当前线程初始化SimpleDateFormat，然后重用该实例。

**我们将这种技术称为线程限制，因为我们将每个实例的使用限制在一个特定的线程中**。

还有另外两种方法可以解决同一问题：

- 使用synchronized块或ReentrantLock
- 按需创建SimpleDateFormat的废弃实例 

不推荐这两种方法：前者在争用较高时会导致显著的性能损失，而后者会创建大量对象，给垃圾回收带来压力。

值得一提的是，**从Java 8开始，引入了一个新的[DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter)类**。这个新的DateTimeFormatter类是不可变的，**并且是线程安全的**。如果我们使用的是Java 8或更高版本，建议使用新的DateTimeFormatter类。

## 3. 解析日期

SimpleDateFormat和DateFormat不仅允许我们格式化日期，还可以进行逆向操作。使用parse方法，**我们可以输入日期的字符串表示形式并返回等效的Date对象**：

```java
SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy");
Date myDate = new Date(233276400000L);
Date parsedDate = formatter.parse("24-05-1977");
assertEquals(myDate.getTime(), parsedDate.getTime());
```

这里需要注意的是，**构造函数中提供的模式应该与使用parse方法解析的日期格式相同**。

## 4. 日期时间模式

SimpleDateFormat在格式化日期时提供了大量不同的选项，完整的列表可以在[JavaDocs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/text/SimpleDateFormat.html)中找到，我们先来了解一下一些比较常用的选项：

| 字母 |日期组件 |     例子|
|:--:|:---:| :----------: |
| M  |  月  |  12; Dec|
| y  |  年  |      94|
| d  |  天  | 	23; Mon|
| H  |  时  |      03|
| m  |  分  |      57|

**日期组件返回的输出很大程度上取决于字符串中使用的字符数**，例如，我们以六月为例，如果我们将日期字符串定义为：

```text
"MM"
```

那么我们的结果将显示为数字代码06，但是，如果我们在日期字符串中添加另一个M：

```text
"MMM"
```

然后，我们得到的格式化日期将显示为单词Jun。

## 5. 应用区域设置

SimpleDateFormat类还支持多种语言环境，这些语言环境是在调用构造函数时设置的。

让我们通过用法语格式化日期来实践一下，我们将实例化一个SimpleDateFormat对象，并在构造函数中传入Locale.FRANCE参数。

```java
SimpleDateFormat franceDateFormatter = new SimpleDateFormat("EEEEE dd-MMMMMMM-yyyy", Locale.FRANCE);
Date myFriday = new Date(1539341312904L);
assertTrue(franceDateFormatter.format(myFriday).startsWith("vendredi"));
```

通过提供一个给定日期(星期三下午)，我们可以断言franceDateFormatter已正确格式化该日期。新的日期正确地以Vendredi(法语中“星期五”的意思)开头。

值得注意的是，[构造函数的Locale版本](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/text/SimpleDateFormat.html#(java.lang.String,java.util.Locale))中存在一个小问题-**虽然支持多种语言环境，但并不能保证完全覆盖**，Oracle建议在DateFormat类中使用工厂方法来确保语言环境覆盖。

## 6. 更改时区

由于SimpleDateFormat扩展了DateFormat类，我们还可以**使用setTimeZone方法来操作时区**。让我们来看看实际的例子：

```java
Date now = new Date();

SimpleDateFormat simpleDateFormat = new SimpleDateFormat("EEEE dd-MMM-yy HH:mm:ssZ");

simpleDateFormat.setTimeZone(TimeZone.getTimeZone("Europe/London"));
logger.info(simpleDateFormat.format(now));

simpleDateFormat.setTimeZone(TimeZone.getTimeZone("America/New_York"));
logger.info(simpleDateFormat.format(now));
```

在上面的示例中，我们在同一个SimpleDateFormat对象上为两个不同的时区提供了相同的日期。我们还在**模式字符串的末尾添加了“Z”字符，以指示时区差异**。然后，format方法的输出会被记录下来供用户查看。

点击运行，我们可以看到相对于两个时区的当前时间：

```text
INFO: Friday 12-Oct-18 12:46:14+0100
INFO: Friday 12-Oct-18 07:46:14-0400
```

## 7. 总结

在本教程中，我们深入探讨了SimpleDateFormat的复杂性。

我们研究了如何实例化SimpleDateFormat以及模式字符串如何影响日期的格式化。