---
layout: post
title:  将java.util.Date转换为java.sql.Date
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 简介

在这个简短的教程中，我们将探讨将java.util.Date转换为java.sql.Date的几种策略。

首先，我们来看一下标准转换，然后，我们将检查一些被认为是最佳实践的替代方案。

## 2. java.util.Date与java.sql.Date

这两个日期类均用于特定场景，并且属于不同的Java标准包：

- [java.util](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/package-summary.html)包是JDK的一部分，包含各种实用程序类以及日期和时间功能。
- [java.sql](https://docs.oracle.com/en/java/javase/21/docs/api/java.sql/java/sql/Date.html)包是JDBC API的一部分，从Java 7开始默认包含在JDK中。

[java.util.Date](https://docs.oracle.com/en/java/javase/21/docs/api/java.sql/java/sql/Date.html)表示特定的时间点，精度为毫秒：

```java
java.util.Date date = new java.util.Date(); 
System.out.println(date);
// Wed Mar 27 08:22:02 IST 2015
```

[java.sql.Date](https://docs.oracle.com/en/java/javase/21/docs/api/java.sql/java/sql/Date.html)是一个对毫秒值的薄包装，允许JDBC驱动程序将其识别为SQL DATE值。此类的值只不过是从[Unix纪元](https://www.baeldung.com/java-date-unix-timestamp)开始以毫秒为单位计算的特定日期的年、月、日，任何比日期更精细的时间信息都将被截断：

```java
long millis=System.currentTimeMillis(); 
java.sql.Date date = new java.sql.Date(millis); 
System.out.println(date);
// 2015-03-30
```

## 3. 为什么需要转换

虽然java.util.Date的用法更为通用，但java.sql.Date用于实现Java应用程序与数据库的通信。因此，在这些情况下需要转换为java.sql.Date。

显式[引用转换](https://www.baeldung.com/java-type-casting)也行不通，因为我们处理的是完全不同的类层次结构：**没有向下转换或向上转换的功能**。如果我们尝试将其中一个日期转换为另一个日期，则会抛出[ClassCastException](https://www.baeldung.com/java-classcastexception#:~:text=ClassCastException%20is%20an%20unchecked%20exception,how%20we%20can%20avoid%20them.)异常：

```java
java.sql.Date date = (java.sql.Date) new java.util.Date() // not allowed
```

## 4. 如何转换为java.sql.Date

有几种将java.util.Date转换 为java.sql.Date的策略，我们将在下面进行探讨。

### 4.1 标准转换

如上所示，java.util.Date包含时间信息，而java.sql.Date不包含。因此，我们可以**使用java.sql.Date的构造函数方法实现有损转换**，该方法接收从Unix纪元开始以毫秒为单位的输入时间：

```java
java.util.Date utilDate = new java.util.Date();
java.sql.Date sqlDate = new java.sql.Date(utilDate.getTime());
```

事实上，丢失表示值的时间部分可能会**导致由于时区不同而报告不同的日期**：

```java
SimpleDateFormat isoFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
isoFormat.setTimeZone(TimeZone.getTimeZone("America/Los_Angeles"));

java.util.Date date = isoFormat.parse("2010-05-23T22:01:02");
TimeZone.setDefault(TimeZone.getTimeZone("America/Los_Angeles"));
java.sql.Date sqlDate = new java.sql.Date(date.getTime());
System.out.println(sqlDate);
// This will print 2010-05-23

TimeZone.setDefault(TimeZone.getTimeZone("Rome"));
sqlDate = new java.sql.Date(date.getTime());
System.out.println(sqlDate);
// This will print 2010-05-24
```

出于这个原因，我们可能需要考虑我们将在下一小节中讨论的其中一种转换替代方案。

### 4.2 使用java.sql.Timestamp代替java.sql.Date

首先要考虑的替代方案是使用java.sql.Timestamp类而不是java.sql.Date，该类也包含有关时间的信息：

```java
java.sql.Timestamp timestamp = new java.sql.Timestamp(date.getTime());
System.out.println(date); //Mon May 24 07:01:02 CEST 2010
System.out.println(timestamp); //2010-05-24 07:01:02.0
```

当然，如果我们查询具有DATE类型的数据库列，则此解决方案可能不是正确的解决方案。

### 4.3 使用java.time包中的类

第二个也是最好的选择是将这两个类都转换为[java.time](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/package-summary.html)包中提供的新类，**此转换的唯一先决条件是使用JDBC 4.2(或更高版本)**，JDBC 4.2于2014年3月随[Java SE 8](https://www.baeldung.com/java-8-new-features)一起发布。

从Java 8开始，我们不再鼓励使用早期Java版本中提供的日期时间类，而是推荐使用新版java.time包中提供的日期时间类。这些增强的类可以更好地满足所有日期/时间需求，包括通过[JDBC驱动程序](https://www.baeldung.com/java-jdbc)与数据库通信。

如果我们采用这种策略，**java.util.Date应该转换为java.time.Instant**：

```java
Date date = new java.util.Date();
Instant instant = date.toInstant().atZone(ZoneId.of("Rome");
```

并且**java.sql.Date应该转换为java.time.LocalDate**：

```java
java.sql.Date sqlDate = new java.sql.Date(timeInMillis);
java.time.LocalDate localDate = sqlDate.toLocalDate();
```

java.time.Instant类可用于映射SQL DATETIME列，java.time.LocalDate可用于映射SQL DATE列。

作为示例，我们现在生成一个带有时区信息的java.util.Date：

```java
SimpleDateFormat isoFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
isoFormat.setTimeZone(TimeZone.getTimeZone("America/Los_Angeles"));
Date date = isoFormat.parse("2010-05-23T22:01:02");
```

接下来，让我们从java.util.Date生成一个LocalDate：

```java
TimeZone.setDefault(TimeZone.getTimeZone("America/Los_Angeles"));
java.time.LocalDate localDate = date.toInstant().atZone(ZoneId.of("America/Los_Angeles")).toLocalDate();
Asserts.assertEqual("2010-05-23", localDate.toString());
```

如果我们尝试切换默认时区，LocalDate将保持相同的值：

```java
TimeZone.setDefault(TimeZone.getTimeZone("Rome"));
localDate = date.toInstant().atZone(ZoneId.of("America/Los_Angeles")).toLocalDate();
Asserts.assertEqual("2010-05-23", localDate.toString())
```

由于我们在转换过程中明确引用了时区，因此一切按预期进行。

## 5. 关于将java.sql.Date转换为java.util.Date

我们已经学习了如何将java.util.Date转换为java.sql.Date。有时，我们只有一个java.sql.Date或java.sql.Timestamp实例，但我们想将其转换为java.util.Date。

实际上，我们不需要将java.sql.Date转换为java.util.Date，这是因为**java.sql.Date和java.sql.Timestamp都是java.util.Date的子类**。因此，**java.sql.Date或java.sql.Timestamp都是java.util.Date**。

接下来我们通过测试来验证一下：

```java
java.util.Date date = UtilToSqlDateUtils.createAmericanDate("2010-05-23T00:00:00");
java.sql.Date sqlDate = new java.sql.Date(date.getTime());
Assertions.assertEquals(date, sqlDate);

java.util.Date dateWithTime = UtilToSqlDateUtils.createAmericanDate("2010-05-23T23:59:59");
java.sql.Timestamp sqlTimestamp = new java.sql.Timestamp(dateWithTime.getTime());
Assertions.assertEquals(dateWithTime, sqlTimestamp);
```

但是，在某些情况下，我们希望**从给定的java.sqlDate或java.sql.Timestamp实例创建一个新的java.util.Date对象**。为此，我们可以利用java.sql.Date或java.sql.Timestamp类的getTime()方法来创建一个新的java.util.Date对象：

```java
java.util.Date date = UtilToSqlDateUtils.createAmericanDate("2010-05-23T00:00:00");
java.sql.Date sqlDate = new java.sql.Date(date.getTime());
java.util.Date newDate = new Date(sqlDate.getTime());
Assertions.assertEquals(date, newDate);

java.util.Date dateWithTime = UtilToSqlDateUtils.createAmericanDate("2010-05-23T23:59:59");
java.sql.Timestamp sqlTimestamp = new java.sql.Timestamp(dateWithTime.getTime());
java.util.Date newDateWithTime = new Date(sqlTimestamp.getTime());
Assertions.assertEquals(dateWithTime, newDateWithTime);
```

## 6. 总结

在本教程中，我们了解了如何将标准的java.util Date转换为java.sql包中提供的Date，除了标准转换之外，我们还研究了两种替代方案；第一种使用Timestamp，第二种依赖于较新的java.time类。