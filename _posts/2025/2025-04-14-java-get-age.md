---
layout: post
title:  用Java计算年龄
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本快速教程中，我们将了解如何使用Java 8、Java 7和Joda-Time库计算年龄。

在很多情况下，我们都会将出生日期和当前日期作为输入，并返回计算出的年龄(以年为单位)。

## 2. 使用Java 8

Java 8引入了[新的Date-Time API](https://www.baeldung.com/migrating-to-java-8-date-time-api)用于处理日期和时间，主要基于Joda-Time库。

在Java 8中，我们可以使用java.time.LocalDate作为我们的出生日期和当前日期，然后使用Period计算它们的年差：

```java
public int calculateAge(LocalDate birthDate, LocalDate currentDate) {
    // validate inputs ...
    return Period.between(birthDate, currentDate).getYears();
}
```

**[LocalDate](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/LocalDate.html)在这里很有用，因为它只表示一个日期**，而Java的Date类则同时表示日期和时间。LocalDate.now()可以返回当前日期。

当我们需要以年、月、日为单位来思考时间段时，[Period](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/Period.html)会很有帮助。

**如果我们想要获得更精确的年龄，比如以秒为单位，那么我们需要分别查看LocalDateTime和Duration(返回一个Long整数)**。

## 3. 使用Joda-Time

如果Java 8不是一个选项，**我们仍然可以从[Joda-Time](http://www.joda.org/joda-time/)获得相同类型的结果**，Joda-Time是Java 8之前日期时间操作的事实标准。

我们需要将[Joda-Time依赖](https://mvnrepository.com/artifact/joda-time/joda-time)添加到pom中：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10</version>
</dependency>
```

然后我们可以编写一个类似的方法来计算年龄，这次使用[Joda-Time](https://www.baeldung.com/joda-time)中的[LocalDate](http://www.joda.org/joda-time/apidocs/index.html)和[Years](http://joda-time.sourceforge.net/apidocs/org/joda/time/Years.html)：

```java
public int calculateAgeWithJodaTime(org.joda.time.LocalDate birthDate, org.joda.time.LocalDate currentDate) {
    // validate inputs ...
    Years age = Years.yearsBetween(birthDate, currentDate);
    return age.getYears();   
}
```

## 4. 使用Java 7

由于Java 7中没有专用的API，我们只能自己动手，因此有相当多的方法。

举个例子，我们可以使用[java.util.Date](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Date.html)：

```java
public int calculateAgeWithJava7(Date birthDate, Date currentDate) {            
    // validate inputs ...                                                                               
    DateFormat formatter = new SimpleDateFormat("yyyyMMdd");                           
    int d1 = Integer.parseInt(formatter.format(birthDate));                            
    int d2 = Integer.parseInt(formatter.format(currentDate));                          
    int age = (d2 - d1) / 10000;                                                       
    return age;                                                                        
}
```

在这里，我们将给定的birthDate和currentDate对象转换为整数，并找出它们之间的差值，**只要8000年后我们还不使用Java 7，这种方法就应该一直有效**。

## 5. 总结

在本文中，我们展示了如何使用Java 8、Java 7和Joda-Time库轻松计算年龄。

要了解有关Java 8日期时间支持的更多信息，请查看[Java 8日期时间简介](https://www.baeldung.com/java-8-date-time-intro)。