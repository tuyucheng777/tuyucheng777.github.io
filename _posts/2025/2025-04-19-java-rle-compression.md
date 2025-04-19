---
layout: post
title:  Java中的游程编码和解码
category: algorithms
copyright: algorithms
excerpt: 游程
---

## 1. 概述

在计算机科学中，数据压缩技术在优化存储和传输效率方面发挥着重要作用，其中一项经受住了时间考验的技术是[游程编码](https://en.wikipedia.org/wiki/Run-length_encoding)(RLE)。

在本教程中，我们将了解RLE并探索如何在Java中实现编码和解码。

## 2. 理解游程编码

游程编码(RLE)是一种简单而有效的无损数据压缩方法，**RLE的基本思想是将数据流中连续的相同元素(称为“run”)用单个值及其计数来表示，而不是用原始的run来表示**。

这在处理重复序列时特别有用，因为它显著减少了存储或传输数据所需的空间量。

RLE非常适合压缩基于调色板的位图图像，例如计算机图标和动画。一个著名的例子是Microsoft Windows 3.x的启动画面，它就是采用RLE压缩的。

让我们考虑以下示例：

```java
String INPUT = "WWWWWWWWWWWWBAAACCDEEEEE";
```

如果我们对上述字符串应用游程编码数据压缩算法，则可以呈现如下效果：

```java
String RLE = "12W1B3A2C1D5E";
```

在编码序列中，**每个字符都遵循其连续出现的次数**，这条规则使得我们在解码过程中可以轻松地重建原始数据。

值得注意的是，**标准RLE仅适用于“文本”输入**。如果输入包含数字，RLE无法以无歧义的方式对其进行编码。

在本教程中，我们将探讨两种游程编码和解码方法。

## 3. 基于字符数组的解决方案

现在我们了解了RLE及其工作原理，实现游程编码和解码的经典方法是将输入字符串转换为char数组，然后将编码和解码规则应用于该数组。

### 3.1 创建编码方法

**实现游程编码的关键是识别每个游程并计算其长度**，我们先来看一下具体实现，然后了解它的工作原理：

```java
String runLengthEncode(String input) {
    StringBuilder result = new StringBuilder();
    int count = 1;
    char[] chars = input.toCharArray();
    for (int i = 0; i < chars.length; i++) {
        char c = chars[i];
        if (i + 1 < chars.length && c == chars[i + 1]) {
            count++;
        } else {
            result.append(count).append(c);
            count = 1;
        }
    }
    return result.toString();
}
```

接下来，我们快速浏览一下上面的代码，并理解其逻辑。

首先，我们使用[StringBuilder](https://www.baeldung.com/java-strings-concatenation#string-builder)存储每个步骤的结果并将它们拼接起来以获得更好的性能。

初始化计数器并将输入字符串转换为字符数组后，该函数将遍历输入字符串中的每个字符。

对于每个字符：

- **如果当前字符与下一个字符相同并且我们不在字符串的末尾**，则计数会增加。
- **如果当前字符与下一个字符不同，或者我们位于字符串的末尾**，则计数和当前字符将附加到结果StringBuilder中。然后，对于下一个唯一字符，计数将重置为1。

最后，使用toString()将StringBuilder转换为字符串并作为编码过程的结果返回。

当我们用这种编码方法测试我们的输入时，我们得到了预期的结果：

```java
assertEquals(RLE, runLengthEncode(INPUT));
```

### 3.2 创建解码方法

识别每个run对于解码RLE字符串仍然至关重要，**run包含字符及其出现的次数，例如“12W”或“2C”**。

现在，让我们创建一个解码方法：

```java
String runLengthDecode(String rle) {
    StringBuilder result = new StringBuilder();
    char[] chars = rle.toCharArray();

    int count = 0;
    for (char c : chars) {
        if (Character.isDigit(c)) {
            count = 10 * count + Character.getNumericValue(c);
        } else {
            result.append(String.join("", Collections.nCopies(count, String.valueOf(c))));
            count = 0;
        }
    }
    return result.toString();
}
```

接下来，让我们分解代码并逐步了解其工作原理。

首先，我们创建一个StringBuilder对象来保存步骤结果，并将rle字符串转换为char数组以供后续处理。

此外，我们初始化了一个整数变量count来跟踪连续发生的次数。

然后，我们遍历RLE编码字符串中的每个字符，对于每个字符：

- **如果字符是数字，则它计入计数**。计数的更新方法是，使用公式10 * count+ Character.getNumericValue(c)将数字附加到现有计数，**这样做是为了处理多位数字计数**。
- **如果字符不是数字，则遇到一个新字符**。然后使用[Collections.nCopies()](https://www.baeldung.com/java-string-of-repeated-characters#1-string-joinand-the-ncopies-methods)将当前字符附加到结果StringBuilder中，[重复count次](https://www.baeldung.com/java-string-of-repeated-characters)。然后，对于下一组连续出现的字符，将计数重置为0。

值得注意的是，**如果我们使用Java 11或更高版本，[String.valueOf(c).repeat(count)](https://www.baeldung.com/java-string-of-repeated-characters#the-jdk11-stringrepeat-function)是重复字符c count次的更好选择**。

当我们使用我们的示例验证解码方法时，测试通过：

```java
assertEquals(INPUT, runLengthDecode(RLE));
```

## 4. 基于正则表达式的解决方案

[正则表达式](https://www.baeldung.com/regular-expressions-java)是处理字符和字符串的强大工具，让我们看看能否使用正则表达式进行游程编码和解码。

### 4.1 创建编码方法

我们先来看输入的字符串，如果能把它[拆分](https://www.baeldung.com/java-split-string)成一个“run数组”，那么问题就迎刃而解了：

```text
Input     : "WWWWWWWWWWWWBAAACCDEEEEE"
Run Array : ["WWWWWWWWWWWW", "B", "AAA", "CC", "D", "EEEEE"]
```

我们不能通过按字符分割输入字符串来实现这一点，相反，我们必须按0宽度位置分割，例如'W'和'B'、'B'和'A'之间的位置，等等。

这些位置的规则并不难发现：**位置前后的字符不同**。因此，我们可以构建一个正则表达式，使用[环视](https://www.baeldung.com/java-regex-lookahead-lookbehind)和后向引用来匹配所需的位置：“(?<=(\\\\D))(?!\\\\1)”，让我们快速理解一下这个正则表达式的含义：

- (?<=(\\\\D))：正向后视断言确保**匹配位于非数字字符之后**(\\\\D表示非数字字符)。
- (?!\\\\1)：负向前视断言确保**匹配的位置不在与正向后视相同的字符之前**，\\\\1指的是先前匹配的非数字字符。

这些断言的组合确保了分割发生在同一字符的连续run之间的边界上。

接下来，让我们创建编码方法：

```java
String runLengthEncodeByRegEx(String input) {
    String[] arr = input.split("(?<=(\\D))(?!\\1)");
    StringBuilder result = new StringBuilder();
    for (String run : arr) {
        result.append(run.length()).append(run.charAt(0));
    }
    return result.toString();
}
```

我们可以看到，在获得run数组之后，剩下的任务就是将每个run的长度和字符附加到准备好的StringBuilder中。

runLengthEncodeByRegEx()方法通过了我们的测试：

```java
assertEquals(RLE, runLengthEncodeByRegEx(INPUT));
```

### 4.2 创建解码方法

我们可以按照类似的思路来解码RLE编码的字符串，首先，我们需要拆分编码后的字符串，得到以下数组：

```text
RLE String: "12W1B3A2C1D5E"
Array     : ["12", "W", "1", "B", "3", "A", "2", "C", "1", "D", "5", "E"]
```

一旦我们得到这个数组，我们就可以通过轻松重复每个字符来生成解码的字符串，例如“W”重复12次，“B”重复1次等。

我们将再次使用环视技术来创建正则表达式来拆分输入字符串：“(?<=\\\\D)|(?=\\\\D+)”。

在这个正则表达式中：

- (?<=\\\\D)：正向后视断言确保拆分发生在非数字字符之后。
- |：表示“或”关系。
- (?=\\\\D+)：正向前视断言确保拆分发生在一个或多个非数字字符之前。

**这种组合允许在RLE编码字符串中的连续计数和字符之间的边界处进行分割**。

接下来，让我们基于正则表达式的拆分来构建解码方法：

```java
String runLengthDecodeByRegEx(String rle) {
    if (rle.isEmpty()) {
        return "";
    }
    String[] arr = rle.split("(?<=\\D)|(?=\\D+)");
    if (arr.length % 2 != 0) {
        throw new IllegalArgumentException("Not a RLE string");
    }
    StringBuilder result = new StringBuilder();

    for (int i = 1; i <= arr.length; i += 2) {
        int count = Integer.parseInt(arr[i - 1]);
        String c = arr[i];

        result.append(String.join("", Collections.nCopies(count, c)));
    }
    return result.toString();
}
```

如代码所示，我们在方法开头添加了一些简单的验证。然后，在迭代拆分数组时，我们从数组中检索计数和字符，并将重复count次的字符附加到结果StringBuilder中。

最后，此解码方法适用于我们的例子：

```java
assertEquals(INPUT, runLengthDecodeByRegEx(RLE));
```

## 5. 总结

在本文中，我们首先讨论了游程编码的工作原理，然后探讨了实现游程编码和解码的两种方法。