---
layout: post
title:  Java Stream API中的DistinctBy
category: libraries
copyright: libraries
excerpt: StreamEx、Vavr
---

## 1. 概述

在列表中搜索不同的元素是我们作为程序员通常面临的常见任务之一，从Java 8开始，随着Stream的加入，我们有了一个新的API来使用函数式方法处理数据。

在本文中，我们将展示使用列表中对象的特定属性过滤集合的不同替代方法。

## 2. 使用Stream API

Stream API提供了distinct()方法，该方法基于Object类的equals()方法返回列表的不同元素。

但是，如果我们想按特定属性进行过滤，它就会变得不那么灵活。我们的替代方案之一是编写一个维护状态的过滤器。

### 2.1 使用有状态过滤器

其中一个可能的解决方案是实现一个有状态的Predicate：

```java
public static <T> Predicate<T> distinctByKey(
        Function<? super T, ?> keyExtractor) {

    Map<Object, Boolean> seen = new ConcurrentHashMap<>();
    return t -> seen.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
}
```

为了测试它，我们将使用以下具有属性age、email和name的Person类：

```java
public class Person { 
    private int age; 
    private String name; 
    private String email; 
    // standard getters and setters 
}
```

要按name获取新的过滤集合，我们可以使用：

```java
List<Person> personListFiltered = personList.stream() 
    .filter(distinctByKey(p -> p.getName())) 
    .collect(Collectors.toList());
```

## 3. 使用Eclipse Collections

[Eclipse Collections](https://www.eclipse.org/collections/)是一个库，它提供了用于处理Java中的流和集合的额外方法。

### 3.1 使用ListIterate.distinct()

ListIterate.distinct()方法允许我们使用各种HashingStrategies来过滤Stream，这些策略可以使用Lambda表达式或方法引用来定义。

如果我们想按name过滤：

```java
List<Person> personListFiltered = ListIterate
    .distinct(personList, HashingStrategies.fromFunction(Person::getName));
```

或者，如果我们要使用的属性是基本类型属性(int、long、double)，我们可以使用这样的专用函数：

```java
List<Person> personListFiltered = ListIterate.distinct(personList, HashingStrategies.fromIntFunction(Person::getAge));
```

### 3.2 Maven依赖

我们需要将以下依赖添加到pom.xml中，以便在我们的项目中使用Eclipse Collections：

```xml
<dependency> 
    <groupId>org.eclipse.collections</groupId> 
    <artifactId>eclipse-collections</artifactId> 
    <version>8.2.0</version> 
</dependency>
```

可以在[Maven中央仓库](https://mvnrepository.com/artifact/org.eclipse.collections/eclipse-collections)中找到最新版本的Eclipse Collections库。

要了解有关此库的更多信息，可以参阅[这篇文章](https://www.baeldung.com/eclipse-collections)。

## 4. 使用Vavr

这是Java 8的函数库，提供不可变数据和函数控制结构。

### 4.1 使用List.distinctBy

为了过滤列表，此类提供了自己的List类，它具有distinctBy()方法，允许我们按其包含的对象的属性进行过滤：

```java
List<Person> personListFiltered = List.ofAll(personList)
    .distinctBy(Person::getName)
    .toJavaList();
```

### 4.2 Maven依赖

我们将在pom.xml中添加以下依赖，以便在项目中使用Vavr。

```xml
<dependency> 
    <groupId>io.vavr</groupId> 
    <artifactId>vavr</artifactId> 
    <version>0.9.0</version>  
</dependency>
```

可以在[Maven中央仓库](https://mvnrepository.com/search?q=vavr)中找到最新版本的Vavr库。

要了解有关此库的更多信息，可以参阅[这篇文章](https://www.baeldung.com/vavr)。

## 5. 使用StreamEx

该库为Java 8流处理提供了有用的类和方法。

### 5.1 使用StreamEx.distinct

在提供的类中，StreamEx具有不同的方法，我们可以向其发送对我们想要区分的属性的引用：

```java
List<Person> personListFiltered = StreamEx.of(personList)
    .distinct(Person::getName)
    .toList();
```

### 5.2 Maven依赖

我们将在pom.xml中添加以下依赖，以便在项目中使用StreamEx。

```xml
<dependency> 
    <groupId>one.util</groupId> 
    <artifactId>streamex</artifactId> 
    <version>0.6.5</version> 
</dependency>
```

可以在[Maven中央仓库](https://mvnrepository.com/artifact/one.util/streamex)中找到最新版本的StreamEx库。

## 6. 总结

在本快速教程中，我们探讨了如何基于使用标准Java 8 API的属性和其他库的其他替代方法获取Stream的不同元素的示例。