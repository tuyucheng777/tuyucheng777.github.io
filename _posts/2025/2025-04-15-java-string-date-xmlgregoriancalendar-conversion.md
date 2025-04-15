---
layout: post
title:  在Java中将字符串日期转换为XMLGregorianCalendar
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将研究将字符串日期转换为XMLGregorianCalendar的各种方法。

## 2. XMLGregorianCalendar

XML Schema标准定义了以XML格式指定日期的明确规则，为了使用此格式，Java 1.5中引入了[XMLGregorianCalendar](https://www.baeldung.com/java-gregorian-calendar)类，它代表了[W3C XML Schema 1.0日期/时间数据类型](https://www.w3.org/TR/xmlschema-2/#isoformats)。

javax.xml.datatype包中的DatatypeFactory类提供了工厂方法，用于创建各种XML Schema内置类型的实例，我们将使用这个类来生成一个新的XMLGregorianCalendar实例。

## 3. 字符串日期转为XMLGregorianCalendar

首先，我们来看看如何将不带时间戳的字符串日期转换为XMLGregorianCalendar格式，日期通常使用yyyy-MM-dd格式表示。

### 3.1 使用标准DatatypeFactory

下面是使用DatatypeFactory将日期字符串解析为XMLGregorianCalendar的示例：

```java
XMLGregorianCalendar usingDatatypeFactoryForDate(String dateString) throws DatatypeConfigurationException {
    return DatatypeFactory.newInstance().newXMLGregorianCalendar(dateString);
}
```

**在上面的例子中，newXMLGregorianCalendar()方法根据日期的字符串表示形式创建了一个XMLGregorianCalendar实例，我们提供的日期遵循XML Schema中的dateTime数据类型**。

让我们通过执行转换来创建XMLGregorianCalendar的实例：

```java
void givenStringDate_whenUsingDatatypeFactory_thenConvertToXMLGregorianCalendar() throws DatatypeConfigurationException {
    String dateAsString = "2014-04-24";
    XMLGregorianCalendar xmlGregorianCalendar = StringDateToXMLGregorianCalendarConverter.usingDatatypeFactoryForDate(dateAsString);
    assertEquals(24, xmlGregorianCalendar.getDay());
    assertEquals(4, xmlGregorianCalendar.getMonth());
    assertEquals(2014, xmlGregorianCalendar.getYear());
}
```

### 3.2 使用LocalDate

**[LocalDate](https://www.baeldung.com/java-8-date-time-intro)是一个不可变、线程安全的类。此外，LocalDate只能保存日期值，而不能包含时间部分**。在这种方法中，我们首先将字符串日期转换为LocalDate实例，然后再将其转换为XMLGregorianCalendar：

```java
XMLGregorianCalendar usingLocalDate(String dateAsString) throws DatatypeConfigurationException {
    LocalDate localDate = LocalDate.parse(dateAsString);
    return DatatypeFactory.newInstance().newXMLGregorianCalendar(localDate.toString());
}
```

我们来看下面的测试代码：

```java
void givenStringDateTime_whenUsingApacheCommonsLang3_thenConvertToXMLGregorianCalendar() throws DatatypeConfigurationException {
    XMLGregorianCalendar xmlGregorianCalendar = StringDateToXMLGregorianCalendarConverter.usingLocalDate(dateAsString);
    assertEquals(24, xmlGregorianCalendar.getDay());
    assertEquals(4, xmlGregorianCalendar.getMonth());
    assertEquals(2014, xmlGregorianCalendar.getYear());
}
```

## 4. 字符串日期和时间转为XMLGregorianCalendar

现在，我们将了解将带有时间戳的字符串日期转换为XMLGregorianCalendar的多种方法，yyyy-MM-dd'T'HH:mm:ss模式通常用于在XML中表示日期和时间。

### 4.1 使用SimpleDateFormat类

将带时间戳的日期转换为XMLGregorianCalendar的传统方法之一是使用[SimplеDatеFormat](https://www.baeldung.com/java-datetimeformatter)类，让我们从dateTimeAsString创建一个XMLGregorianCalendar实例：

```java
XMLGregorianCalendar usingSimpleDateFormat(String dateTimeAsString) throws DatatypeConfigurationException, ParseException {
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
    Date date = simpleDateFormat.parse(dateTimeAsString);
    return DatatypeFactory.newInstance().newXMLGregorianCalendar(simpleDateFormat.format(date));
}
```

我们使用SimpleDateFormat将输入的String dateTime解析为Date对象，然后在创建XMLGregorianCalendar实例之前将Date对象格式化回String。

让我们测试一下这种方法：

```java
void givenStringDateTime_whenUsingSimpleDateFormat_thenConvertToXMLGregorianCalendar() throws DatatypeConfigurationException, ParseException {
    XMLGregorianCalendar xmlGregorianCalendar = StringDateToXMLGregorianCalendarConverter.usingSimpleDateFormat(dateTimeAsString);
    assertEquals(24, xmlGregorianCalendar.getDay());
    assertEquals(4, xmlGregorianCalendar.getMonth());
    assertEquals(2014, xmlGregorianCalendar.getYear());
    assertEquals(15, xmlGregorianCalendar.getHour());
    assertEquals(45, xmlGregorianCalendar.getMinute());
    assertEquals(30, xmlGregorianCalendar.getSecond());
}
```

### 4.2 使用GregorianCalendar类

[GregorianCalendar](https://www.baeldung.com/java-gregorian-calendar)是抽象类java.util.Calendar的具体实现，让我们使用GregorianCalendar类将String日期和时间转换为XMLGregorianCalendar：

```java
XMLGregorianCalendar usingGregorianCalendar(String dateTimeAsString) throws DatatypeConfigurationException, ParseException {
    GregorianCalendar calendar = new GregorianCalendar();
    calendar.setTime(new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss").parse(dateTimeAsString));
    return DatatypeFactory.newInstance().newXMLGregorianCalendar(calendar);
}
```

首先，我们创建一个GregorianCalendar实例，并根据解析后的Date设置其时间。之后，我们使用DatatypeFactory创建一个XMLGregorianCalendar实例。让我们测试一下这种方法：

```java
void givenStringDateTime_whenUsingGregorianCalendar_thenConvertToXMLGregorianCalendar() throws DatatypeConfigurationException, ParseException {
    XMLGregorianCalendar xmlGregorianCalendar = StringDateToXMLGregorianCalendarConverter.usingGregorianCalendar(dateTimeAsString);
    assertEquals(24, xmlGregorianCalendar.getDay());
    assertEquals(4, xmlGregorianCalendar.getMonth());
    assertEquals(2014, xmlGregorianCalendar.getYear());
    assertEquals(15, xmlGregorianCalendar.getHour());
    assertEquals(45, xmlGregorianCalendar.getMinute());
    assertEquals(30, xmlGregorianCalendar.getSecond());
}
```

### 4.3 使用Joda-Time

[Joda-Time](https://www.baeldung.com/joda-time)是Java中流行的日期和时间操作库，它为标准Java日期和时间API提供了更直观的替代方案。

让我们探索如何使用Joda-Time将字符串日期和时间转换为XMLGregorianCalendar：

```java
XMLGregorianCalendar usingJodaTime(String dateTimeAsString) throws DatatypeConfigurationException {
    DateTime dateTime = DateTime.parse(dateTimeAsString, DateTimeFormat.forPattern("yyyy-MM-dd'T'HH:mm:ss"));
    return DatatypeFactory.newInstance().newXMLGregorianCalendar(dateTime.toGregorianCalendar());
}
```

这里，我们根据提供的dateTimeAsString值实例化了DatеTime对象，然后使用toGregorianCalendar()方法将此dateTime对象转换为GregorianCalendar实例。最后，我们使用DatatypeFactory类的newXMLGregorianCalendar()方法创建了一个XMLGregorianCalendar实例。

让我们测试一下这种方法：

```java
void givenStringDateTime_whenUsingJodaTime_thenConvertToXMLGregorianCalendar() throws DatatypeConfigurationException {
    XMLGregorianCalendar xmlGregorianCalendar = StringDateToXMLGregorianCalendarConverter.usingJodaTime(dateTimeAsString);
    assertEquals(24, xmlGregorianCalendar.getDay());
    assertEquals(4, xmlGregorianCalendar.getMonth());
    assertEquals(2014, xmlGregorianCalendar.getYear());
    assertEquals(15, xmlGregorianCalendar.getHour());
    assertEquals(45, xmlGregorianCalendar.getMinute());
    assertEquals(30, xmlGregorianCalendar.getSecond());
}
```

## 5. 总结

在本快速教程中，我们讨论了将字符串日期转换为XMLGregorianCalendar实例的各种方法。