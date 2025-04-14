---
layout: post
title:  如何设置JVM时区
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

我们的应用程序用户对时间戳的要求可能很高，他们希望我们的应用程序能够自动检测他们的时区，并以正确的时区显示时间戳。

**在本教程中，我们将介绍几种修改JVM时区的方法**，我们还将了解与管理时区相关的一些陷阱。

## 2. 时区介绍

默认情况下，JVM从操作系统读取时区信息，**该信息会传递给[TimeZone](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TimeZone.html)类，该类存储时区信息并计算夏令时**。

我们可以调用getDefault方法，它将返回程序运行时的时区。此外，我们可以使用TimeZone.getAvailableIDs()从应用程序获取支持的时区ID列表。

在命名时区时，Java依赖于[tz数据库](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List)的命名约定。

## 3. 更改时区

在本节中，我们将介绍几种更改JVM中时区的方法。

### 3.1 设置环境变量

我们先来看看如何使用环境变量来更改时区，我们可以添加或修改环境变量TZ。

例如，在基于Linux的环境中，我们可以使用export命令：

```shell
export TZ="America/Sao_Paulo"
```

设置环境变量后，我们可以看到正在运行的应用程序的时区现在是America/Sao_Paulo：

```java
Calendar calendar = Calendar.getInstance();
assertEquals(calendar.getTimeZone(), TimeZone.getTimeZone("America/Sao_Paulo"));
```

### 3.2 设置JVM参数

**设置环境变量的另一种方法是设置JVM参数user.timezone**，此JVM参数优先于环境变量TZ。

例如，我们可以在运行应用程序时使用标志-D：

```shell
java -Duser.timezone="Asia/Kolkata" com.company.Main
```

**同样，我们也可以从应用程序中设置JVM参数**：

```java
System.setProperty("user.timezone", "Asia/Kolkata");
```

我们现在可以看到时区是Asia/Kolkata：

```java
Calendar calendar = Calendar.getInstance();
assertEquals(calendar.getTimeZone(), TimeZone.getTimeZone("Asia/Kolkata"));
```

### 3.3 从应用程序设置时区

**最后，我们还可以使用TimeZone类从应用程序中修改JVM时区**，此方法优先于环境变量和JVM参数。

设置默认时区很简单：

```java
TimeZone.setDefault(TimeZone.getTimeZone("Portugal"));
```

正如预期的那样，现在的时区是Portugal：

```java
Calendar calendar = Calendar.getInstance();
assertEquals(calendar.getTimeZone(), TimeZone.getTimeZone("Portugal"));
```

## 4. 陷阱

### 4.1 使用三个字母的时区ID

尽管可以使用三个字母的ID来表示时区，但[不建议这样做](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/TimeZone.html)。

**相反，我们应该使用更长的名称，因为三个字母的ID容易产生歧义**。例如，IST可能是印度标准时间、爱尔兰标准时间或以色列标准时间。

### 4.2 全局设置

请注意，上述每种方法都是为整个应用程序全局设置时区。然而，在现代应用程序中，时区的设置通常比这更加细致。

例如，我们可能需要将时间转换为最终用户的时区，因此全局时区意义不大。如果不需要全局时区，可以考虑在每个日期时间实例上直接指定时区，[ZonedDateTime或OffsetDateTime](https://www.baeldung.com/java-zoneddatetime-offsetdatetime)类非常适合这种情况。

## 5. 总结

在本教程中，我们解释了几种修改JVM时区的方法，可以设置系统范围的环境变量，更改JVM参数，或者从应用程序中以编程方式修改它。