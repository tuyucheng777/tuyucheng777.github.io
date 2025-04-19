---
layout: post
title:  屏蔽字符串(最后N个字符除外)
category: algorithms
copyright: algorithms
excerpt: OpenPDF
---

## 1. 字符串

在Java中，我们经常需要屏蔽一个字符串，例如隐藏在日志文件中打印或显示给用户的敏感信息。

在本教程中，我们将探讨如何使用一些简单的Java技术来实现这一点。

## 2. 问题介绍

在许多情况下，我们可能需要屏蔽敏感信息，例如信用卡号、社保号，甚至电子邮件地址，一种常见的做法是隐藏字符串中除最后几个字符之外的所有字符。

假设我们有3个敏感的字符串值：

```java
static final String INPUT_1 = "a b c d 1234";
static final String INPUT_2 = "a b c d     ";
static final String INPUT_3 = "a b";
```

现在，我们要屏蔽这些字符串值中除最后一个N之外的所有字符，为简单起见，**本教程中我们取N=4，并使用星号(\*)屏蔽每个字符**。因此，预期结果如下：

```java
static final String EXPECTED_1 = "********1234";
static final String EXPECTED_2 = "********    ";
static final String EXPECTED_3 = "a b";
```

如我们所见，**如果输入字符串的长度小于或等于N(4)，我们将跳过对其进行屏蔽**。此外，我们将空格字符与常规字符同等对待。

接下来，我们将以这些字符串输入为例，使用不同的方法来屏蔽它们，以获得预期的结果。像往常一样，我们将利用单元测试断言来验证每个解决方案是否正常工作。

## 3. 使用char数组

我们知道字符串是由一系列字符组成的，因此，**我们可以将字符串输入转换为字符数组，然后将屏蔽逻辑应用于字符数组**：

```java
String maskByCharArray(String input) {
    if (input.length() <= 4) {
        return input;
    }
    char[] chars = input.toCharArray();
    Arrays.fill(chars, 0, chars.length - 4, '*');
    return new String(chars);
}
```

接下来，让我们逐步了解其实现过程并了解其工作原理。

首先，我们检查输入的长度是否小于或等于4。如果是，则不应用屏蔽，并按原样返回输入字符串。

然后，我们使用toCharArray()方法将输入字符串转换为char[\]类型。接下来，**我们利用[Arrays.fill()](https://www.baeldung.com/java-initialize-array#adding-values-to-arrays-using-arraysfill)来屏蔽字符**。Arrays.fill()允许我们定义输入char[\]的哪个部分需要用特定字符填充，在本例中，我们只想使用fill()屏蔽chars.length - 数组开头4个字符。

最后，在屏蔽char数组的所需部分后，我们使用new String(chars)[将其转换回字符串](https://www.baeldung.com/java-char-array-to-string)并返回结果。

接下来我们测试一下这个解决方案是否能达到预期的效果：

```java
assertEquals(EXPECTED_1, maskByCharArray(INPUT_1));
assertEquals(EXPECTED_2, maskByCharArray(INPUT_2));
assertEquals(EXPECTED_3, maskByCharArray(INPUT_3));
```

测试表明，它正确地屏蔽了我们的3个输入。

## 4. 两个子字符串

我们的需求是屏蔽给定字符串中除最后4个字符之外的所有字符，换句话说，**我们可以将输入字符串分成两个子字符串：一个需要屏蔽的子字符串(toMask)，以及一个包含最后4个字符的子字符串(keepPlain)**。

然后，我们可以简单地将toMask子字符串中的所有字符替换为“\*”，并将两个子字符串拼接在一起以形成最终结果：

```java
String maskBySubstring(String input) {
    if (input.length() <= 4) {
        return input;
    }
    String toMask = input.substring(0, input.length() - 4);
    String keepPlain = input.substring(input.length() - 4);
    return toMask.replaceAll(".", "*") + keepPlain;
}
```

如代码所示，与字符数组方法类似，我们首先处理输入的length() <= 4的情况。

然后我们使用[substring()](https://www.baeldung.com/string/substring)方法提取两个子字符串：toMask和keepPlain。

我们调用[replaceAll()](https://www.baeldung.com/string/replace-all/)通过将任意字符“.”替换为“\*”来屏蔽toMask，需要注意的是，**这里的“.”参数是一个正则表达式(regex)，用于匹配任意字符，而不是字面意义上的句点字符**。

最后，我们将屏蔽部分与未屏蔽部分(keepPlain)拼接起来并返回结果。

该方法也通过了我们的测试：

```java
assertEquals(EXPECTED_1, maskBySubstring(INPUT_1));
assertEquals(EXPECTED_2, maskBySubstring(INPUT_2));
assertEquals(EXPECTED_3, maskBySubstring(INPUT_3));
```

我们可以看到，这种方法可以简洁明了地解决该问题。

## 5. 使用正则表达式

在两个子字符串的解决方案中，我们使用了replaceAll()。我们还提到过，此方法支持正则表达式。实际上，通过使用replaceAll()和一个巧妙的正则表达式，我们可以一步高效地解决这个掩码问题。

接下来，让我们看看这是如何做到的：

```java
String maskByRegex(String input) {
    if (input.length() <= 4) {
        return input;
    }
    return input.replaceAll(".(?=.{4})", "*");
}
```

在此示例中，除了input.length() <= 4的情况处理外，**我们仅通过一次replaceAll()调用来应用屏蔽逻辑**。接下来，让我们理解其中的魔法，即正则表达式“.(?=.{4})”。

这是一个[前向断言](https://www.baeldung.com/java-regex-lookahead-lookbehind/)，它确保只有紧随其后的4个字符才不会被屏蔽。

简而言之，**正则表达式会查找后跟4个字符(.{4})的任意字符(.)，并将其替换为“*”，这样可以确保只屏蔽最后4个字符之前的字符**。

如果我们用输入进行测试，我们会得到预期的结果：

```java
assertEquals(EXPECTED_1, maskByRegex(INPUT_1));
assertEquals(EXPECTED_2, maskByRegex(INPUT_2));
assertEquals(EXPECTED_3, maskByRegex(INPUT_3));
```

正则表达式方法可以在一次传递中有效地处理屏蔽，使其成为简洁代码的理想选择。

## 6. 使用repeat()方法

从Java 11开始，[repeat()](https://www.baeldung.com/java-string-of-repeated-characters#the-jdk11-stringrepeat-function)方法加入到String类中，它允许我们通过重复某个字符多次来创建一个String值，如果我们使用Java 11或更高版本，我们可以使用repeat()来解决屏蔽问题：

```java
String maskByRepeat(String input) {
    if (input.length() <= 4) {
        return input;
    }
    int maskLen = input.length() - 4;
    return "*".repeat(maskLen) + input.substring(maskLen);
}
```

在这个方法中，我们首先计算屏蔽长度(maskLen)：input.length() - 4。然后，**我们直接重复屏蔽字符所需的长度，并将其与未屏蔽的子字符串拼接起来，形成最终结果**。

像往常一样，让我们使用输入字符串来测试这种方法：

```java
assertEquals(EXPECTED_1, maskByRepeat(INPUT_1));
assertEquals(EXPECTED_2, maskByRepeat(INPUT_2));
assertEquals(EXPECTED_3, maskByRepeat(INPUT_3));
```

## 7. 总结

在本文中，我们探讨了屏蔽字符串同时保持最后4个字符可见的不同方法。

值得注意的是，虽然我们在本教程中选择了“*”字符来屏蔽敏感信息并保持最后4个(N = 4)个字符可见，但这些方法可以轻松调整以适应不同的要求，例如，不同的屏蔽字符或N值。