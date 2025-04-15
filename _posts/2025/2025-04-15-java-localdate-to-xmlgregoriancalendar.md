---
layout: post
title:  LocalDate和XMLGregorianCalendar之间的转换
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本快速教程中，我们将讨论LocalDate和XMLGregorianCalendar，并提供两种类型之间转换的示例。

## 2. XMLGregorianCalendar

XML Schema标准定义了以XML格式指定日期的明确规则，为了使用此格式，Java 1.5中引入了[XMLGregorianCalendar](https://www.baeldung.com/java-gregorian-calendar)类，它是[W3C XML Schema 1.0日期/时间数据类型](https://www.w3.org/TR/xmlschema-2/#isoformats)的表示。

## 3. LocalDate

[LocalDate](https://www.baeldung.com/java-8-date-time-intro)实例表示ISO-8601日历系统中不带时区的日期，因此，LocalDate适合存储生日等数据，但不适合存储任何与时间相关的数据。Java在1.8版本中引入了LocalDate。

## 4. 从LocalDate到XMLGregorianCalendar

首先，我们来看看如何将LocalDate转换为XMLGregorianCalendar。为了生成一个新的XMLGregorianCalendar实例，我们使用了javax.xml.datatype包中的DataTypeFactory。

因此，让我们创建一个LocalDate实例并将其转换为XMLGregorianCalendar：

```java
LocalDate localDate = LocalDate.of(2019, 4, 25);

XMLGregorianCalendar xmlGregorianCalendar = DatatypeFactory.newInstance().newXMLGregorianCalendar(localDate.toString());

assertThat(xmlGregorianCalendar.getYear()).isEqualTo(localDate.getYear());
assertThat(xmlGregorianCalendar.getMonth()).isEqualTo(localDate.getMonthValue());
assertThat(xmlGregorianCalendar.getDay()).isEqualTo(localDate.getDayOfMonth());
assertThat(xmlGregorianCalendar.getTimezone()).isEqualTo(DatatypeConstants.FIELD_UNDEFINED);
```

如前所述，XMLGregorianCalendar实例可能包含时区信息。然而，LocalDate不包含任何时间信息。

因此，当我们执行转换时，**时区值将保持为FIELD_UNDEFINED**。

## 5. 从XMLGregorianCalendar到LocalDate

同样，我们现在来看看如何反过来进行转换。事实证明，从XMLGregorianCalendar转换为LocalDate要容易得多。

同样，由于LocalDate没有有关时间的信息，因此LocalDate实例只能包含XMLGregorianCalendar信息的子集。

让我们创建一个XMLGregorianCalendar实例并执行转换：

```java
XMLGregorianCalendar xmlGregorianCalendar = 
  DatatypeFactory.newInstance().newXMLGregorianCalendar("2019-04-25");

LocalDate localDate = LocalDate.of(
    xmlGregorianCalendar.getYear(), 
    xmlGregorianCalendar.getMonth(), 
    xmlGregorianCalendar.getDay());

assertThat(localDate.getYear()).isEqualTo(xmlGregorianCalendar.getYear());
assertThat(localDate.getMonthValue()).isEqualTo(xmlGregorianCalendar.getMonth());
assertThat(localDate.getDayOfMonth()).isEqualTo(xmlGregorianCalendar.getDay());
```

## 6. 总结

在本快速教程中，我们介绍了LocalDate实例和XMLGregorianCalendar之间的转换。