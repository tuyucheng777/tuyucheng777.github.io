---
layout: post
title:  找出字符串中的最大数字
category: algorithms
copyright: algorithms
excerpt: 字符串
---

## 1. 简介

通常，在处理多个编程场景时，会出现包含[数字](https://www.baeldung.com/java-number-class)的[字符串](https://www.baeldung.com/java-string)，并且可能需要在这些值中找到最大值。

**在本教程中，我们将深入研究不同的方法和Java代码说明，以便正确识别和提取给定字符串中的最大数值**。

## 2. 字符串解析与比较

最简单的方法是读取字符串并识别数字子串，我们可以通过比较前缀来检测最大的数字，举个例子：

```java
String inputString = "The numbers are 10, 20, and 5";
int expectedLargestNumber = 20;

@Test
void givenInputString_whenUsingBasicApproach_thenFindingLargestNumber() {
    String[] numbers = inputString.split("[^0-9]+");

    int largestNumber = Integer.MIN_VALUE;
    for (String number : numbers) {
        if (!number.isEmpty()) {
            int currentNumber = Integer.parseInt(number);
            if (currentNumber > largestNumber) {
                largestNumber = currentNumber;
            }
        }
    }
    assertEquals(expectedLargestNumber, largestNumber);
}
```

这里，我们首先使用split()方法将名为inputString的输入字符串拆分为一个子字符串数组，这种拆分通过正则表达式[^0-9\]+进行，该正则表达式仅截取字符串中的数字。

随后，一个常规循环演示了字符串拆分。该循环限制数组仅包含生成的子字符串，并且故意不包含空字符串，**每个非空子字符串的实现都包含一个使用Integer.parseInt()方法的显著转换**。

**然后，将当前数值与迄今为止找到的biggestNumber进行比较，如果遇到更大的值，则进行更新**。最后，我们使用assertEquals()方法来确保biggestNumber等于expectedLargestNumber。

## 3. 使用正则表达式进行高效的数字提取

[正则表达式](https://www.baeldung.com/regular-expressions-java)使我们能够简洁有效地从字符串中提取数值，利用Pattern和Matcher类，我们可以使这个过程更加简化，这是一个简单的例子：

```java
@Test
void givenInputString_whenUsingRegularExpression_thenFindingLargestNumber() {
    Pattern pattern = Pattern.compile("\\d+");
    Matcher matcher = pattern.matcher(inputString);

    int largestNumber = Integer.MIN_VALUE;
    while (matcher.find()) {
        int currentNumber = Integer.parseInt(matcher.group());
        if (currentNumber > largestNumber) {
            largestNumber = currentNumber;
        }
    }
    assertEquals(expectedLargestNumber, largestNumber);
}
```

这里，我们首先使用Pattern.compile()方法编译一个正则表达式(\\d+)，该表达式经过精心设计，专注于匹配输入字符串中的一个或多个数字。

然后，我们通过将编译的模式应用于inputString来初始化Matcher对象(表示为matcher)。

**之后，我们进入一个while循环。每次迭代中，都会使用Integer.parseInt(matcher.group())方法提取数值。然后，会展开一个关键的比较过程，将当前数值与现有的biggestNumber进行比较，如果发现更大的值，largestNumber会立即更新以反映这一识别**。

## 4. Stream和Lambda表达式

Java 8提出了[Stream API](https://www.baeldung.com/java-stream-filter-Lambda)和Lambda表达式，使得代码更加紧凑，更加易读。

我们来简单实现一下：

```java
@Test
void givenInputString_whenUsingStreamAndLambdaExpression_thenFindingLargestNumber() {
    int largestNumber = Arrays.stream(inputString.split("[^0-9]+"))
            .filter(s -> !s.isEmpty())
            .mapToInt(Integer::parseInt)
            .max()
            .orElse(Integer.MIN_VALUE);

    assertEquals(expectedLargestNumber, largestNumber);
}
```

在这个测试方法中，我们首先过滤字符串，提取其中的数字部分，这通过使用split()方法实现。此外，我们还采取了一些措施来应对可能出现的空流，并实现了isEmpty()方法。

**初始过滤之后，我们利用mapToInt()方法，通过Integer::parseInt引用，系统地将每个非空子字符串转换为整数。随后，max()操作可以有效地识别处理流中存在的最大整数值**。

我们采用orElse()方法来包装简化的方法，策略性地将默认值设置为Integer.MIN_VALUE。

## 5. 总结

总之，本教程全面介绍了一些技术，使在Java中处理包含数字的字符串变得更加容易。