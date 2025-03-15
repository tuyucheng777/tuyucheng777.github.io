---
layout: post
title:  Mockito中的泛型List匹配器
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在使用[Mockito](https://www.baeldung.com/mockito-series)编写Java单元测试时，我们经常需要对接收[泛型](https://www.baeldung.com/java-generics)List参数的方法进行存根，例如List<String\>、List<Integer\>等。但是，由于Java的类型擦除，处理这些情况需要额外考虑。

在本教程中，我们将探索如何在Mockito中使用带有泛型的List匹配器，我们将介绍Java 7和Java 8+的情况。

## 2. 问题介绍

首先，我们通过一个例子来了解泛型和Mockito挑战。

假设我们有一个带有默认方法的接口：

```java
interface MyInterface {
    default String extractFirstLetters(List<String> words) {
        return String.join(", ", words.stream().map(str -> str.substring(0, 1)).toList());
    }
}
```

extractFirstLetters()方法接收一个字符串值列表，并返回一个以逗号分隔的字符串，其中包含列表中每个单词的第一个字符。

现在，我们想在测试中MockMyInterface并存根extractFirstLetters()方法：

```java
MyInterface mock = Mockito.mock(MyInterface.class);
when(mock.extractFirstLetters(any(List.class))).thenReturn("a, b, c, d, e");
assertEquals("a, b, c, d, e", mock.extractFirstLetters(new ArrayList<String>()));
```

在此示例中，我们仅使用[ArgumentMatchers](https://www.baeldung.com/mockito-argument-matchers).any(List.class)来匹配泛型List参数，如果我们运行测试，它会通过。因此，存根按预期工作。

但是，如果我们检查编译器日志，我们会看到一个警告：

```text
Unchecked assignment: 'java.util.List' to 'java.util.List<java.lang.String>' 
```

这是因为我们使用了any(List.class)来匹配泛型List<String\>参数，编译器无法在编译时验证原始List是否仅包含String元素。

接下来，让我们探索存根方法和匹配泛型List参数的正确方法。由于Java 7和Java 8+中类型推断的处理方式不同，因此我们将同时介绍Java 7和Java 8+的情况。

## 3. Java 7中匹配泛型列表参数

有时，我们必须处理使用旧Java版本的遗留Java项目。

在Java 7中，类型推断受到限制，因此当使用anyList()之类的ArgumentMatchers时，编译器很难确定正确的泛型类型。因此，在使用Mockito的ArgumentMatchers时，**我们必须指定泛型类型**。

接下来，让我们看看如何在Java 7中对extractFirstLetters()方法进行存根：

```java
//Java7
MyInterface mock = Mockito.mock(MyInterface.class);
when(mock.extractFirstLetters(ArgumentMatchers.<String>anyList())).thenReturn("a, b, c, d, e");
assertEquals("a, b, c, d, e", mock.extractFirstLetters(new ArrayList<>()));
```

如测试所示，**我们在anyList()匹配器上指定了<String\>类型**。测试编译并通过。

如果没有明确指定<String\>，anyList()将返回List<?\>，这与预期的List<String\>不匹配，并 导致Java 7中的编译器错误。

## 4. 在Java 8+中匹配泛型列表参数

Java 8及以后的版本中，编译器变得更加智能，我们不需要显式指定类型参数。

因此，**我们可以简单地使用anyList()而不指定<String\>**，编译器就会正确推断出预期的类型：

```java
MyInterface mock = Mockito.mock(MyInterface.class);
when(mock.extractFirstLetters(anyList())).thenReturn("a, b, c, d, e");
assertEquals("a, b, c, d, e", mock.extractFirstLetters(new ArrayList<>()));
```

如果我们执行测试，它会通过并且不会出现编译器警告。这是因为**Java 8+编译器会自动从extractFirstLetters(List<String\>)推断出泛型类型**。

某些人可能会想出一种使用any()匹配器来匹配所需的泛型参数的方法来：

```java
MyInterface mock = Mockito.mock(MyInterface.class);
when(mock.extractFirstLetters(any())).thenReturn("a, b, c, d, e");
assertEquals("a, b, c, d, e", mock.extractFirstLetters(new ArrayList<>()));
```

代码看起来紧凑而直观。同样，Java 8+编译器可以从目标方法推断出泛型类型，因此这种方法同样有效。但是，**any()是一个匹配任何对象的泛型匹配器**。它不太特定于类型，并且可能导致匹配器不太精确的情况。

实际上，对于明确接收List<String\>的方法，使用anyList()会更精确、更自文档化，清楚地表明匹配器需要List。因此，尽管从技术上讲两种匹配都可以使用，但在处理List参数时，**为了获得更好的类型安全性和可读性，最好使用anyList()**。

## 5. 总结

Mockito的List匹配器可以轻松使用泛型List参数，但我们需要注意类型擦除和Java的类型推断。

在本文中，我们探讨了如何正确匹配泛型List参数：

- 在Java 7中–我们必须明确指定泛型类型：ArgumentMatchers.<T\>anyList()
- 在Java及更高版本中-我们可以直接使用anyList()而无需明确指定<T\>，因为编译器可以自动推断类型

理解这些概念使我们能够在传统和现代Java项目中编写更清晰、更有效的单元测试。