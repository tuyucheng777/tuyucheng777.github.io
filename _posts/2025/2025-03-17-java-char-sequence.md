---
layout: post
title:  Java中的增量字符
category: java
copyright: java
excerpt: Java Char
---

## 1. 概述

在本教程中，我们将学习如何在Java中生成从“A”到“Z”的字符序列，我们将通过自增ASCII值来实现这一点。

在Java中，我们用Unicode表示ASCII值，因为ASCII字符的范围很广。标准ASCII表限制为127个字符，Unicode包含的字符比ASCII多得多，可以实现国际化和使用各种符号。因此，在Java中，我们不仅限于ASCII的标准值。

我们将使用Java 8 [Stream API](https://www.baeldung.com/java-streams)中的IntStream和[for](https://www.baeldung.com/java-for-loop)循环生成一系列字符。

## 2. 使用for循环

我们将使用标准for循环创建从'A'到'Z'的大写字母列表：

```java
@Test 
void whenUsingForLoop_thenGenerateCharacters(){
    List<Character> allCapitalCharacters = Arrays.asList('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z');
  
    List<Character> characters = new ArrayList<>();
    for (char character = 'A'; character <= 'Z'; character++) {
        characters.add(character);   
    }
  
    Assertions.assertEquals(alphabets, allCapitalCharacters);
}
```

每个字母在ASCII系统中都有一个唯一的数字，例如，“A”表示为65，“B”表示为66，一直到最后的“Z”表示为90。

在上面的例子中，我们自增了char类型并将其添加到for循环中的列表中。

最后，通过使用Assertions类的assertEquals()方法，我们检查生成的列表是否与所有大写字符的预期列表匹配。

## 3. 使用Java 8 IntStream

使用Java 8 IntStream，我们可以生成从'A'到'Z'的所有大写字母序列：

```java
@Test
void whenUsingStreams_thenGenerateCharacters() {
    List<Character> allCapitalCharacters = Arrays.asList('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z');

    List<Character> characters = IntStream.rangeClosed('A', 'Z')
        .mapToObj(c -> (char) c)
        .collect(Collectors.toList());
    Assertions.assertEquals(characters, allCapitalCharacters);
}
```

在上面的例子中，使用Java 8中的IntStream，我们生成从“A”到“Z”的字符，其ASCII值范围为65到90。

首先，我们将这些值映射到字符，然后将它们收集到列表中。

IntStream接收两个整数作为参数，但通过传递一个字符作为参数，编译器会自动将其转换为整数。如果我们将其转换为整数列表，我们将看到有一个从65到90的整数列表。

`IntStream.rangeClosed('A', 'Z')`

最后，我们使用Assertions类中的assertEquals()方法来验证生成的列表是否与所有大写字母的预期列表一致。

## 4. 总结

在这篇短文中，我们探讨了如何使用Stream API和for循环来自增字符的ASCII值并打印其对应的值。