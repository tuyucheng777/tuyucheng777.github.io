---
layout: post
title:  在Java中使用当前时间作为文件名
category: java-date
copyright: java-date
excerpt: Java Date
---

##  1. 简介

在本教程中，我们将介绍几种在Java中获取当前时间戳值并将其用作文件名的方法。

为了实现我们的目标，我们将利用[Java DateTime API](https://www.baeldung.com/java-8-date-time-intro)和第三方库(如[Joda-Time](https://www.baeldung.com/joda-time))中的几个类。

## 2. 初始设置

在后面的部分中，我们将构建几个测试用例，展示获取当前时间戳并将其用作文件名的每种方法。

但是，要将时间戳值转换为指定的字符串格式，我们首先需要指定时间戳格式，然后使用它来定义格式化程序类：

```java
static final String TIMESTAMP_FORMAT = "yyyyMMddHHmmss";
static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern(TIMESTAMP_FORMAT);
static final SimpleDateFormat SIMPLEDATE_FORMAT = new SimpleDateFormat(TIMESTAMP_FORMAT);
```

接下来，我们将编写一个方法，将当前时间值转换为有效的文件名，此方法的示例输出将类似于“20231209122307.txt”：

```java
String getFileName(String currentTime) {
    return MessageFormat.format("{0}.txt", currentTime);
}
```

由于我们需要编写测试用例，我们将创建另一种方法来检查输出文件名是否包含具有正确格式的时间戳：

```java
boolean verifyFileName(String fileName) {
    return Pattern
        .compile("[0-9]{14}+\\.txt", Pattern.CASE_INSENSITIVE)
        .matcher(fileName)
        .matches();
}
```

在这种情况下，我们的文件名由代表时间戳的数字组成，**建议确保文件名格式避免使用特定操作系统禁用的字符**。

## 3. 使用Java DateTime API获取当前时间

Java提供了诸如Calendar和Date之类的遗留类来处理日期和时间信息，然而，由于设计缺陷，Java 8的DateTime API引入了一些新类。Date、Calendar和SimpleDateFormatter类是可变的，并且不是线程安全的。

我们首先看一下传统的Calendar和Date类来生成时间戳并获得基本的了解，**然后了解Java 8 DateTime API类，如Instant、LocalDateTime、ZonedDateTime和OffsetDateTime**。

**值得注意的是，对于较新的Java程序，建议使用Java 8 DateTime API类，而不是旧版Java日期和时间类**。

### 3.1 使用Calendar

最基本的方法是使用Calendar.getInstance()方法，该方法返回一个使用默认时区和语言环境的[Calendar](https://www.baeldung.com/java-gregorian-calendar)实例。此外，getTime()方法返回以毫秒为单位的时间值：

```java
@Test
public void whenUsingCalender_thenGetCurrentTime() {
    String currentTime = SIMPLEDATE_FORMAT.format(Calendar.getInstance().getTime());
    String fileName = getFileName(currentTime);
  
    assertTrue(verifyFileName(fileName));
}
```

[SimpleDateFormatter](https://www.baeldung.com/java-simple-date-format)类可以将时间值转换为适当的时间戳格式。

### 3.2 使用Date

类似地，我们可以构造一个Date类的对象，以毫秒为单位表示对象的创建时间。SimpleDateFormatter将毫秒时间值转换为所需的字符串模式：

```java
@Test
public void whenUsingDate_thenGetCurrentTime() {
    String currentTime = SIMPLEDATE_FORMAT.format(new Date());
    String fileName = getFileName(currentTime);
  
    assertTrue(verifyFileName(fileName));
}
```

**建议使用我们将在下一节中看到的新Java 8类**。

### 3.3 使用Instant

在Java中，Instant类代表UTC时间轴上的单个时刻：

```java
@Test
public void whenUsingInstant_thenGetCurrentTime() {
    String currentTime = Instant
        .now()
        .truncatedTo(ChronoUnit.SECONDS)
        .toString()
        .replaceAll("[:TZ-]", "");
    String fileName = getFileName(currentTime);

    assertTrue(verifyFileName(fileName));
}
```

Instant.now()方法向系统时钟查询当前时刻，我们可以使用truncatedTo()方法将该值四舍五入到最接近的秒数。然后，可以将秒数转换为字符串，以替换时间戳中时区信息中任何不需要的字符。

### 3.4 使用LocalDateTime

LocalDateTime表示ISO-8601日历系统中不带时区的日期和时间：

```java
@Test
public void whenUsingLocalDateTime_thenGetCurrentTime() {
    String currentTime = LocalDateTime.now().format(DATETIME_FORMATTER);
    String fileName = getFileName(currentTime);

    assertTrue(verifyFileName(fileName));
}
```

LocalDateTime.now()方法查询默认系统时钟，提供日期时间信息。然后，我们可以传递一个[DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter)将时间戳格式化为字符串。

### 3.5 使用ZonedDateTime

ZonedDateTime是带有时区的日期时间的不可变表示：

```java
@Test
public void whenUsingZonedDateTime_thenGetCurrentTime() {
    String currentTime = ZonedDateTime
        .now(ZoneId.of("Europe/Paris"))
        .format(DATETIME_FORMATTER);
    String fileName = getFileName(currentTime);

    assertTrue(verifyFileName(fileName));
}
```

**时区标识符能够唯一地标识地球上的特定地理位置，例如“Europe/Paris”**，使用此标识，我们可以获取ZoneId，它决定了从Instant到LocalDateTime的转换所使用的时区。

**ZonedDateTime会自动处理全年的夏令时(DST)调整**。

### 3.6 使用OffsetDateTime

OffsetDateTime是ZonedDateTime的简化版本，它忽略了时区。世界不同地区的时区偏移量有所不同，例如，“+2:00”表示某个时区的时间比UTC晚两个小时。我们可以将偏移量与[ZoneOffSet](https://www.baeldung.com/java-zone-offset)结合使用，从而更改默认的UTC时间：

```java
@Test
public void whenUsingOffsetDateTime_thenGetCurrentTime() {
    String currentTime = OffsetDateTime
        .of(LocalDateTime.now(), ZoneOffset.of("+02:00"))
        .format(DATETIME_FORMATTER);
    String fileName = getFileName(currentTime);

    assertTrue(verifyFileName(fileName));
}
```

ZonedDateTime和OffsetDateTime都存储时间轴的瞬间，精度可达纳秒，了解它们的[区别](https://www.baeldung.com/java-zoneddatetime-offsetdatetime)有助于我们在两者之间做出选择。

## 4. 使用Joda-Time获取当前时间

Joda-Time是一个著名的日期和时间处理库，它是开发人员中最流行的库之一，可以替代繁琐的遗留Java类；**它使用不可变类来处理日期和时间值**。

让我们在pom.xml中添加Joda-Time Maven[依赖](https://mvnrepository.com/artifact/joda-time/joda-time)：

```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

### 4.1 使用Joda DateTime

DateTime.now()方法使用默认时区获取设置为当前系统毫秒时间的DateTime，然后，我们可以将其转换为具有定义时间戳格式的字符串：

```java
@Test
public void whenUsingJodaTime_thenGetCurrentTime() {
    String currentTime = DateTime.now().toString(TIMESTAMP_FORMAT);
    String fileName = getFileName(currentTime);

    assertTrue(verifyFileName(fileName));
}
```

### 4.2 使用Joda Instant

Joda-Time库也提供了Instant类来捕获当前时间轴中的时刻，我们可以使用[DateTimeFormat](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)将时间戳转换为所需的字符串模式：

```java
@Test
public void whenUsingJodaTimeInstant_thenGetCurrentTime() {
    String currentTime = DateTimeFormat
        .forPattern(TIMESTAMP_FORMAT)
        .print(org.joda.time.Instant.now().toDateTime());
    String fileName = getFileName(currentTime);

    assertTrue(verifyFileName(fileName));
}
```

## 5. 总结

在本文中，我们探索了在Java程序中获取当前时间戳的多种方法，并利用它们生成文件名，我们使用各种Java DateTime API类和Joda-Time库来获取当前时间戳。