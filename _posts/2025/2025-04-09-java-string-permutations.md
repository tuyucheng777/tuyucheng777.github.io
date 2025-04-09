---
layout: post
title:  Java中字符串的排列
category: algorithms
copyright: algorithms
excerpt: 字符串
---

## 1. 简介

[排列](https://www.baeldung.com/cs/array-generate-all-permutations)是集合中元素的重新排列，换句话说，它是集合顺序的所有可能变化。

在本教程中，我们将学习如何使用第三方库[在Java中轻松创建排列](https://www.baeldung.com/java-array-permutations)，更具体地说，我们将处理字符串中的排列。

## 2. 排列

有时，我们需要检查字符串值的所有可能排列，这通常用于令人费解的在线编程练习，而日常工作中则相对少见。例如，字符串“abc”有6种不同的字符排列方式：“abc”、“acb”、“cab”、“bac”、“bca”、“cba”。

一些定义明确的算法可以帮助我们为特定的字符串值创建所有可能的排列，例如，最著名的是堆算法。然而，它相当复杂且不直观；除此之外，递归方法使情况变得更糟。

## 3. 优雅的解决方案

实现生成排列的算法需要编写自定义逻辑，在实现过程中很容易出错，而且很难测试它是否随着时间的推移正确运行。此外，重写之前写的内容也毫无意义。

此外，在使用字符串值时，如果不小心操作，可能会因创建过多实例而导致字符串池泛滥。

以下是当前提供此类功能的库：

- Apache Commons
- Guava
- CombinatoricsLib

让我们尝试使用这些库来查找字符串值的所有排列，**我们将关注这些库是否允许对排列进行惰性遍历以及它们如何处理输入值中的重复项**。

我们将在下面的示例中使用Helper.toCharacterList方法，此方法封装了将字符串转换为字符列表的复杂性：
```java
static List<Character> toCharacterList(final String string) {
    return string.chars().mapToObj(s -> ((char) s)).collect(Collectors.toList());
}
```

此外，我们将使用辅助方法将字符列表转换为字符串：
```java
static String toString(Collection<Character> collection) {
    return collection.stream().map(s -> s.toString()).collect(Collectors.joining());
}
```

## 4. Apache Commons

首先，让我们将Maven依赖[commons-collections4](https://mvnrepository.com/artifact/org.apache.commons/commons-collections4)添加到项目中：
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.5.0-M2</version>
</dependency>
```

总体而言，Apache提供了一个简单的API，**CollectionUtils会急切地创建排列，因此在处理长字符串值时应小心谨慎**：
```java
public List<String> eagerPermutationWithRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    return CollectionUtils.permutations(characters)
        .stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

**同时，为了使其以惰性方式工作，我们应该使用PermutationIterator**：
```java
public List<String> lazyPermutationWithoutRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    final PermutationIterator<Character> permutationIterator = new PermutationIterator<>(characters);
    final List<String> result = new ArrayList<>();
    while (permutationIterator.hasNext()) {
        result.add(Helper.toString(permutationIterator.next()));
    }
    return result;
}
```

**此库不处理重复项，因此字符串“aaaaaa”将产生720个排列，这通常是不可取的**。此外，PermutationIterator没有获取排列数的方法。在这种情况下，我们应该根据输入大小分别计算它们。

## 5. Guava

首先，让我们将[Guava库](https://mvnrepository.com/artifact/com.google.guava/guava)的Maven依赖添加到项目中：
```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
</dependency>
```

Guava允许使用Collections2创建排列，该API使用起来非常简单：
```java
public List<String> permutationWithRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    return Collections2.permutations(characters).stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

Collections2.permutations的结果是一个PermutationCollection，它允许轻松访问排列，**所有排列都是惰性创建的**。

**此外，该类还提供了用于创建无重复排列的API**：
```java
public List<String> permutationWithoutRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    return Collections2.orderedPermutations(characters).stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

但是，这些方法的问题在于它们被用[@Beta注解](https://guava.dev/releases/18.0/api/docs/com/google/common/annotations/Beta.html)标注，这并不能保证该API在未来版本中不会发生变化。

## 6. CombinatoricsLib

为了在项目中使用它，让我们添加[combinatoricslib3](https://mvnrepository.com/artifact/com.github.dpaukov/combinatoricslib3) Maven依赖：
```xml
<dependency>
    <groupId>com.github.dpaukov</groupId>
    <artifactId>combinatoricslib3</artifactId>
    <version>3.3.3</version>
</dependency>
```

虽然这是一个小型库，但它提供了许多组合工具，包括排列。API本身非常直观，并利用Java Stream。让我们从特定字符串或字符列表创建排列：
```java
public List<String> permutationWithoutRepetitions(final String string) {
    List<Character> chars = Helper.toCharacterList(string);
    return Generator.permutation(chars)
        .simple()
        .stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

上面的代码创建了一个生成器，它将为字符串提供排列。排列将被延迟检索，因此，我们只创建了一个生成器并计算了预期的排列数。

同时，通过这个库，我们可以识别重复项的策略。以字符串“aaaaaa”为例，我们将只得到一个相同的排列，而不是720个。
```java
public List<String> permutationWithRepetitions(final String string) {
    List<Character> chars = Helper.toCharacterList(string);
    return Generator.permutation(chars)
        .simple(TreatDuplicatesAs.IDENTICAL)
        .stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

TreatDuplicatesAs允许我们定义如何处理重复项。

## 7. 总结

有很多方法可以处理组合和排列，所有这些库都可以极大的帮助你。