---
layout: post
title:  在Java中按字母顺序对字符串进行排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 概述

在本教程中，我们将展示如何按字母顺序对字符串进行排序。

我们这样做可能有很多原因-其中之一可能是验证两个单词是否由相同的字符集组成，这样，我们就可以验证它们是否是字谜。

## 2. 对字符串进行排序

在内部，String使用字符数组进行操作。因此，我们可以**使用toCharArray() : char[]方法，对数组进行排序并根据结果创建一个新的String**：
```java
@Test
void givenString_whenSort_thenSorted() {
    String abcd = "bdca";
    char[] chars = abcd.toCharArray();

    Arrays.sort(chars);
    String sorted = new String(chars);

    assertThat(sorted).isEqualTo("abcd");
}
```

在Java 8中，我们可以利用Stream API对字符串进行排序：
```java
@Test
void givenString_whenSortJava8_thenSorted() {
    String sorted = "bdca".chars()
        .sorted()
        .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
        .toString();

    assertThat(sorted).isEqualTo("abcd");
}
```

这里，我们使用与第一个例子相同的算法，但使用Stream sorted()方法对char数组进行排序。

注意，字符是按照其ASCII码排序的，因此大写字母总是出现在开头。因此，如果我们想对“abC”进行排序，排序结果将是“Cab”。

为了解决这个问题，我们需要使用toLowerCase()方法转换字符串，我们将在Anagram验证器示例中执行此操作。

## 3. 测试

为了测试排序，我们将构建一个字谜验证器。如前所述，当两个不同的单词或句子由同一组字符组成时，就会发生字谜。

让我们看一下AnagramValidator类：
```java
public class AnagramValidator {

    public static boolean isValid(String text, String anagram) {
        text = prepare(text);
        anagram = prepare(anagram);

        String sortedText = sort(text);
        String sortedAnagram = sort(anagram);

        return sortedText.equals(sortedAnagram);
    }

    private static String sort(String text) {
        char[] chars = prepare(text).toCharArray();

        Arrays.sort(chars);
        return new String(chars);
    }

    private static String prepare(String text) {
        return text.toLowerCase()
                .trim()
                .replaceAll("\\s+", "");
    }
}
```

现在，我们将利用我们的排序方法并验证字谜是否有效：
```java
@Test
void givenValidAnagrams_whenSorted_thenEqual() {
    boolean isValidAnagram = AnagramValidator.isValid("Avida Dollars", "Salvador Dali");
        
    assertTrue(isValidAnagram);
}
```

## 4. 总结

在这篇简短的文章中，我们展示了如何以两种方式按字母顺序对字符串进行排序。此外，我们还实现了使用字符串排序方法的字谜验证器。