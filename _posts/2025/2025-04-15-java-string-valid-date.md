---
layout: post
title:  在Java中检查字符串是否为有效日期
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在本教程中，我们将讨论在Java中检查字符串是否包含有效日期的各种方法。

我们将研究Java 8之前、Java 8之后以及使用[Apache Commons Validator](http://commons.apache.org/proper/commons-validator/)的解决方案。

## 2. 日期验证概述

无论何时我们在任何应用程序中接收数据，我们都需要验证其有效性，然后再进行进一步处理。

对于日期输入，我们可能需要验证以下内容：

- 输入包含有效格式的日期，例如MM/DD/YYYY
- 输入的各个部分都在有效范围内
- 输入解析为日历中的有效日期

我们可以使用[正则表达式](https://www.baeldung.com/java-date-regular-expressions)来实现上述功能，然而，处理各种输入格式和语言环境的正则表达式非常复杂，容易出错，还会降低性能。

**我们将讨论以灵活、强大和高效的方式实现日期验证的不同方法**。

首先，我们来编写一个日期校验的接口：

```java
public interface DateValidator {
   boolean isValid(String dateStr);
}
```

在接下来的部分中，我们将使用各种方法实现该接口。

## 3. 使用DateFormat进行验证

Java自诞生之日起就提供了格式化和解析日期的功能，此功能包含在[DateFormat](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/text/DateFormat.html)抽象类及其实现[SimpleDateFormat](https://www.baeldung.com/java-simple-date-format)中。

让我们使用DateFormat类的parse方法来实现日期验证：

```java
public class DateValidatorUsingDateFormat implements DateValidator {
    private String dateFormat;

    public DateValidatorUsingDateFormat(String dateFormat) {
        this.dateFormat = dateFormat;
    }

    @Override
    public boolean isValid(String dateStr) {
        DateFormat sdf = new SimpleDateFormat(this.dateFormat);
        sdf.setLenient(false);
        try {
            sdf.parse(dateStr);
        } catch (ParseException e) {
            return false;
        }
        return true;
    }
}
```

由于**DateFormat和相关类不是线程安全的**，因此我们为每个方法调用创建一个新实例。

接下来我们来编写这个类的单元测试：

```java
DateValidator validator = new DateValidatorUsingDateFormat("MM/dd/yyyy");

assertTrue(validator.isValid("02/28/2019"));        
assertFalse(validator.isValid("02/30/2019"));
```

这是Java 8之前最常见的解决方案。

## 4. 使用LocalDate进行验证

Java 8引入了[改进的日期和时间API](https://www.baeldung.com/java-8-date-time-intro)，它添加了[LocalDate](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/LocalDate.html)类，**该类表示不带时间的日期，并且是不可变的且线程安全的**。

LocalDate提供了两种静态方法来解析日期，并且都使用[DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter)进行实际解析：

```java
public static LocalDate parse(CharSequence text)
// parses dates using using DateTimeFormatter.ISO_LOCAL_DATE

public static LocalDate parse(CharSequence text, DateTimeFormatter formatter)
// parses dates using the provided formatter
```

让我们使用parse方法来实现日期验证：

```java
public class DateValidatorUsingLocalDate implements DateValidator {
    private DateTimeFormatter dateFormatter;

    public DateValidatorUsingLocalDate(DateTimeFormatter dateFormatter) {
        this.dateFormatter = dateFormatter;
    }

    @Override
    public boolean isValid(String dateStr) {
        try {
            LocalDate.parse(dateStr, this.dateFormatter);
        } catch (DateTimeParseException e) {
            return false;
        }
        return true;
    }
}
```

该实现使用DateTimeFormatter对象进行格式化，由于该类是线程安全的，因此我们在不同的方法调用中使用同一个实例。

并为这个实现添加一个单元测试：

```java
DateTimeFormatter dateFormatter = DateTimeFormatter.BASIC_ISO_DATE;
DateValidator validator = new DateValidatorUsingLocalDate(dateFormatter);
        
assertTrue(validator.isValid("20190228"));
assertFalse(validator.isValid("20190230"));
```

## 5. 使用DateTimeFormatter进行验证

在上一节中，我们看到LocalDate使用DateTimeFormatter对象进行解析，我们也可以直接使用DateTimeFormatter类进行格式化和解析。

**DateTimeFormatter分两个阶段解析文本**：在第一阶段，它根据配置将文本解析为各种日期和时间字段；在第二阶段，它将解析后的字段解析为日期和/或时间对象。

ResolverStyle属性控制第2阶段，它是一个枚举，具有3个可能的值：

- LENIENT：宽松解析日期和时间
- SMART：以智能方式解析日期和时间
- STRICT：严格解析日期和时间

现在让我们直接使用DateTimeFormatter编写日期验证：

```java
public class DateValidatorUsingDateTimeFormatter implements DateValidator {
    private DateTimeFormatter dateFormatter;

    public DateValidatorUsingDateTimeFormatter(DateTimeFormatter dateFormatter) {
        this.dateFormatter = dateFormatter;
    }

    @Override
    public boolean isValid(String dateStr) {
        try {
            this.dateFormatter.parse(dateStr);
        } catch (DateTimeParseException e) {
            return false;
        }
        return true;
    }
}
```

接下来，我们为这个类添加单元测试：

```java
DateTimeFormatter dateFormatter = DateTimeFormatter.ofPattern("uuuu-MM-dd", Locale.US)
    .withResolverStyle(ResolverStyle.STRICT);
DateValidator validator = new DateValidatorUsingDateTimeFormatter(dateFormatter);
        
assertTrue(validator.isValid("2019-02-28"));
assertFalse(validator.isValid("2019-02-30"));
```

在上面的测试中，我们根据模式和语言环境创建了一个DateTimeFormatter，我们对日期使用了严格的解析。

## 6. 使用Apache Commons Validator进行验证

[Apache Commons](https://commons.apache.org/)项目提供了一个验证框架，**其中包含校验例程，例如日期、时间、数字、货币、IP地址、电子邮件和URL**。

对于本文，让我们看一下[GenericValidator](http://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/GenericValidator.html)类，它提供了几种方法来检查字符串是否包含有效日期：

```java
public static boolean isDate(String value, Locale locale)
  
public static boolean isDate(String value,String datePattern, boolean strict)
```

要使用该库，让我们将[commons-validator](https://mvnrepository.com/artifact/commons-validator/commons-validator) Maven依赖添加到项目中：

```xml
<dependency>
    <groupId>commons-validator</groupId>
    <artifactId>commons-validator</artifactId>
    <version>1.6</version>
</dependency>
```

接下来，让我们使用GenericValidator类来验证日期：

```java
assertTrue(GenericValidator.isDate("2019-02-28", "yyyy-MM-dd", true));
assertFalse(GenericValidator.isDate("2019-02-29", "yyyy-MM-dd", true));
```

## 7. 总结

在本文中，我们研究了检查字符串是否包含有效日期的各种方法。