---
layout: post
title:  Cactoos库指南
category: libraries
copyright: libraries
excerpt: Cactoos
---

## 1. 简介

**[Cactoos](https://github.com/yegor256/cactoos)是一个面向对象的Java基本类型库**。

在本教程中，我们将介绍该库中的一些可用类。

## 2. Cactoos

Cactoos库的功能非常丰富，从字符串操作到数据结构。该库提供的原始类型及其对应的方法与其他库(如[Guava](https://www.baeldung.com/category/guava/)和[Apache Commons](https://www.baeldung.com/java-commons-lang-3))提供的类似，但更侧重于面向对象的[设计原则](https://www.baeldung.com/solid-principles)。

### 2.1 与Apache Commons的比较

Cactoos库配备了提供与作为Apache Commons库一部分的静态方法相同功能的类。

让我们看一下StringUtils包中的一些静态方法及其在Cactoos中的等效类：

| StringUtils的静态方法       | 等效的Cactoos类        |
|:-----------------------|:-------------------|
| isBlank()                  | IsBlank                 |
| lowerCase()                   | Lowered                 |
| upperCase()                   | Upper                  |
| rotate()                   | Rotated                |
| swapCase()                | SwappedCase              |
| stripStart()                 | TrimmedLeft                |
| stripEnd()                 | TrimmedRight                |

有关这方面的更多信息，请参见[官方文档](https://github.com/yegor256/cactoos#our-objects-vs-their-static-methods)。

## 3. Maven依赖

让我们首先添加所需的Maven依赖，这个库的最新版本可以在[Maven Central]https://mvnrepository.com/artifact/org.cactoos/cactoos)上找到：

```xml
<dependency>
    <groupId>org.cactoos</groupId>
    <artifactId>cactoos</artifactId>
    <version>0.43</version>
</dependency>
```

## 4. 字符串

Cactoos有大量用于操作String对象的类。

### 4.1 字符串对象创建

让我们看看如何使用TextOf类创建String对象：

```java
String testString = new TextOf("Test String").asString();
```

### 4.2 格式化字符串

如果需要创建格式化的字符串，我们可以使用FormattedText类：

```java
String formattedString = new FormattedText("Hello %s", stringToFormat).asString();
```

让我们验证此方法实际上返回格式化的String：

```java
StringMethods obj = new StringMethods();

String formattedString = obj.createdFormattedString("John");
assertEquals("Hello John", formattedString);
```

### 4.3 小写/大写字符串

Lowered类使用其TextOf对象将String转换为小写：

```java
String lowerCaseString = new Lowered(new TextOf(testString)).asString();
```

同样，可以使用Upper类将给定的String转换为大写：

```java
String upperCaseString = new Upper(new TextOf(testString)).asString();
```

让我们使用测试字符串验证这些方法的输出：

```java
StringMethods obj = new StringMethods();

String lowerCaseString = obj.toLowerCase("TeSt StrIng");
String upperCaseString = obj.toUpperCase("TeSt StrIng"); 

assertEquals("test string", lowerCaseString);
assertEquals("TEST STRING", upperCaseString);
```

### 4.4 检查空字符串

如前所述，Cactoos库提供了一个IsBlank类来检查null或空String：

```java
new IsBlank(new TextOf(testString)) != null;
```

## 5. 集合

该库还提供了几个用于处理Collections的类，让我们来看看其中的一些。

### 5.1 迭代集合

我们可以使用实用程序类And迭代字符串列表：

```java
new And(
    new Mapped<>(
        new FuncOf<>(
            input -> System.out.printf("Item: %s\n", input),
            new True()
        ),
        strings
    )
).value();
```

上述方法是遍历字符串列表并将输出写入记录器的一种实用方法。

### 5.2 过滤集合

Filtered类可用于根据特定条件过滤集合：

```java
public Collection<String> getFilteredList(List<String> strings) {
    return new ListOf<>(
            new Filtered<>(
                    s -> s.length() == 5,
                    strings
            )
    );
}
```

让我们通过传入几个参数来测试此方法，其中只有3个参数满足条件：

```java
CollectionUtils obj = new CollectionUtils(); 

List<String> strings = new ArrayList<String>() {
    add("Hello"); 
    add("John");
    add("Smith"); 
    add("Eric"); 
    add("Dizzy"); 
};

int size = obj.getFilteredList(strings).size(); 

assertEquals(3, size);
```

该库提供的一些其他的Collections类可以在[官方文档](https://github.com/yegor256/cactoos#iterablescollectionslistssets)中找到。

## 6. 总结

在本教程中，我们了解了Cactoos库及其为字符串和数据结构操作提供的一些类。

除了这些之外，该库还提供了其他用于[IO操作](https://github.com/yegor256/cactoos#inputoutput)以及[日期和时间](https://github.com/yegor256/cactoos#dates-and-times)的实用程序类。