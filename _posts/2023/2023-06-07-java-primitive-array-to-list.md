---
layout: post
title:  将基本类型数组转换为列表
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1.概述

在这个简短的教程中，我们将展示如何将基元数组转换为相应类型的对象列表。通常，我们可能会尝试在Java中使用[自动装箱。](https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)然而，正如我们将在下一节中看到的那样，我们对自动装箱如何工作的直觉常常是错误的。

## 2.问题

让我们从问题的定义开始。我们有一个基元数组(int[])，我们希望将该数组转换为一个列表(List<Integer>)。一个直观的第一次尝试可能是：

```java
int[] input = new int[]{1,2,3,4};
List<Integer> output = Arrays.asList(input);
```

不幸的是，由于类型不兼容，这不会编译。我们可能希望自动装箱能够处理基元数组。然而，这种本能的信念是不正确的。

自动装箱只发生在单个元素上(例如从int到Integer)。没有从基本类型数组到其装箱引用类型数组的自动转换(例如从int[]到Integer[])。

让我们开始针对这个问题实施一些解决方案。

## 3.迭代

由于自动装箱适用于单个基本元素，因此一个简单的解决方案是遍历数组的元素并将它们一一添加到列表中：

```java
int[] input = new int[]{1,2,3,4};
List<Integer> output = new ArrayList<Integer>();
for (int value : input) {
    output.add(value);
}
```

我们已经解决了问题，但是解决方案非常冗长。这将我们带到下一个实现。

## 4.溪流

从Java8开始，我们可以使用[StreamAPI](https://www.baeldung.com/java-streams)。我们可以使用Stream提供单线解决方案：

```java
int[] input = new int[]{1,2,3,4};
List<Integer> output = Arrays.stream(input).boxed().collect(Collectors.toList());
```

或者，我们可以使用IntStream：

```java
int[] input = new int[]{1,2,3,4};
List<Integer> output = IntStream.of(input).boxed().collect(Collectors.toList());
```

这当然看起来好多了。接下来，我们将看看几个外部库。

## 5.番石榴

[Guava](https://www.baeldung.com/category/guava/)库提供了一个解决这个问题的包装器。让我们从添加[Maven](https://search.maven.org/artifact/com.google.guava/guava)依赖项开始：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
    <type>bundle</type>
</dependency>
```

我们可以使用Ints.asList()，其他原始类型有类似的工具类：

```java
int[] input = new int[]{1,2,3,4};
List<Integer> output = Ints.asList(input);
```

## 6.阿帕奇公地

另一个库是[ApacheCommonsLang](https://search.maven.org/artifact/org.apache.commons/commons-lang3)。同样，让我们为这个库添加Maven依赖项：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

更准确地说，我们使用ArrayUtils类：

```java
int[] input = new int[]{1,2,3,4};
Integer[] outputBoxed = ArrayUtils.toObject(input);
List<Integer> output = Arrays.asList(outputBoxed);
```

## 七、总结

现在，我们的工具箱中有几个选项可以将基元数组转换为List。正如我们所见，自动装箱只发生在单个元素上。在本教程中，我们看到了几种应用转换的解决方案。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-1)上获得。