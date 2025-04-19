---
layout: post
title:  将Java字符串旋转N个字符
category: algorithms
copyright: algorithms
excerpt: 字符串
---

## 1. 概述

在日常Java编程中，字符串通常是我们必须处理的基本对象。有时，我们需要将给定的字符串旋转n个字符。旋转字符串需要以循环方式移动其字符，从而产生动态且视觉上引人入胜的效果。

在本教程中，我们将探索解决字符串旋转问题的不同方法。

## 2. 问题介绍

**当我们说将字符串旋转n个字符时，实际上就是将字符串中的n个字符移动**。

### 2.1 例子

假设我们有一个字符串对象：

```java
String STRING = "abcdefg";
```

如果我们以STRING作为输入，旋转n个字符后，结果将如下所示：

```text
- Forward Rotation -
Input String    : abcdefg
Rotate (n = 1) -> gabcdef
Rotate (n = 2) -> fgabcde
Rotate (n = 3) -> efgabcd
Rotate (n = 4) -> defgabc
Rotate (n = 5) -> cdefgab
Rotate (n = 6) -> bcdefga
Rotate (n = 7) -> abcdefg
Rotate (n = 8) -> gabcdef
...
```

上面的例子说明了字符串正向旋转的行为，但是，字符串也可以反向旋转，如下所示：

```text
- Backward Rotation -
Input String    : abcdefg
Rotate (n = 1) -> bcdefga
Rotate (n = 2) -> cdefgab
Rotate (n = 3) -> defgabc
Rotate (n = 4) -> efgabcd
Rotate (n = 5) -> fgabcde
Rotate (n = 6) -> gabcdef
Rotate (n = 7) -> abcdefg
...
```

在本教程中，我们将探索正向旋转和反向旋转。我们的目标是创建一种方法，通过移动n个字符，将输入字符串按指定方向旋转。

为了简单起见，我们将限制我们的方法，使其仅接收n的非负值。

### 2.2 分析问题

在深入研究代码之前，让我们先分析一下这个问题并总结一下它的关键特征。

首先，如果我们通过移动0个(n = 0)个字符来旋转字符串，无论方向如何，结果都应该与输入字符串镜像。这本身就很明显，因为当n等于0时不会发生旋转。

此外，如果我们看一下例子，当n = 7时，输出再次等于输入：

```text
Input String    : abcdefg
...
Rotate (n = 7) -> abcdefg
...
```

出现此现象的原因是，输入字符串的长度为7，当n等于STRING.length时，每个字符在旋转后都会回到原来的位置。因此，**将STRING旋转STRING并移动STRING.length个字符后，其结果与原始STRING完全相同**。

现在，显而易见，**当n = STRING.length × K(其中K为整数)时，输入和输出字符串相等**。简单来说，**移位字符的有效n'本质上等于n % STRING.length**。

接下来，我们来看一下旋转方向。对比之前提供的正向和反向旋转示例，可以看出**“反向旋转n”本质上等同于“正向旋转STRING.length - n”**。例如，n = 2的反向旋转与n = 5(STRING.length - 2)的正向旋转结果相同，如下所示：

```text
- Forward Rotation -
Rotate (n = 5) -> cdefgab
...

- Backward Rotation -
Rotate (n = 2) -> cdefgab
...
```

因此，**我们只能集中解决正向旋转问题，将所有反向旋转转化为正向旋转**。

让我们简要列出到目前为止所学到的内容：

- 有效n' =n % STRING.length
- n = 0或K × STRING.length，结果 = STRING
- “反向旋转n”可以转换为“正向旋转(STRING.length - n)”

### 2.3 准备测试数据

由于我们将使用单元测试来验证我们的解决方案，因此让我们创建一些预期输出来涵盖各种场景：

```java
// forward
String EXPECT_1X = "gabcdef";
String EXPECT_2X = "fgabcde";
String EXPECT_3X = "efgabcd";
String EXPECT_6X = "bcdefga";
String EXPECT_7X = "abcdefg";  // len = 7
String EXPECT_24X = "efgabcd"; //24 = 3 x 7(len) + 3

// backward
String B_EXPECT_1X = "bcdefga";
String B_EXPECT_2X = "cdefgab";
String B_EXPECT_3X = "defgabc";
String B_EXPECT_6X = "gabcdef";
String B_EXPECT_7X = "abcdefg";
String B_EXPECT_24X = "defgabc";
```

接下来我们来看第一个解决方案“拆分和合并”。

## 3. 拆分和合并

解决这个问题的思路是**将输入字符串拆分成两个子字符串，然后交换它们的位置并重新组合**。像往常一样，一个例子可以帮助我们快速理解这个想法。

假设我们要将STRING正向循环移位两个(n = 2)个字符，那么，我们可以按照以下方式执行循环：

```text
Index   0   1   2   3   4   5   6
STRING  a   b   c   d   e   f   g

Sub1   [a   b   c   d   e] -->
Sub2                   <-- [f   g]
Result [f   g] [a   b   c   d   e]
```

因此，解决问题的关键是找到两个子字符串的索引范围，这对我们来说不是什么挑战：

- Sub 1：[0, STRING.length – n)，在这个例子中是[0,5)
- Sub 2：[STRING.length – n, STRING.length)在这个例子中，它是[5,7)

值得注意的是，上面使用的半开符号“[a,b)”表示索引“a”包含在内，而“b”不包含在内。有趣的是，**Java的[String.subString(beginIndex, endIndex)](https://www.baeldung.com/string/substring)方法恰好遵循了同样的约定，排除了endIndex，从而简化了索引计算**。

现在，基于我们的理解，实现起来就变得简单了：

```java
String rotateString1(String s, int c, boolean forward) {
    if (c < 0) {
        throw new IllegalArgumentException("Rotation character count cannot be negative!");
    }
    int len = s.length();
    int n = c % len;
    if (n == 0) {
        return s;
    }
    n = forward ? n : len - n;
    return s.substring(len - n, len) + s.substring(0, len - n);
}
```

如上图所示，布尔变量forward表示预期旋转的方向。随后，我们使用表达式“n = forward ? n : len - n”将反向旋转无缝转换为正向旋转。

此外，该方法成功通过了我们准备的测试用例：

```java
// forward
assertEquals(EXPECT_1X, rotateString1(STRING, 1, true));
assertEquals(EXPECT_2X, rotateString1(STRING, 2, true));
assertEquals(EXPECT_3X, rotateString1(STRING, 3, true));
assertEquals(EXPECT_6X, rotateString1(STRING, 6, true));
assertEquals(EXPECT_7X, rotateString1(STRING, 7, true));
assertEquals(EXPECT_24X, rotateString1(STRING, 24, true));

// backward
assertEquals(B_EXPECT_1X, rotateString1(STRING, 1, false));
assertEquals(B_EXPECT_2X, rotateString1(STRING, 2, false));
assertEquals(B_EXPECT_3X, rotateString1(STRING, 3, false));
assertEquals(B_EXPECT_6X, rotateString1(STRING, 6, false));
assertEquals(B_EXPECT_7X, rotateString1(STRING, 7, false));
assertEquals(B_EXPECT_24X, rotateString1(STRING, 24, false));
```

## 4. 自拼接和提取

这种方法的本质在于将字符串与其自身拼接起来，即SS = STRING + STRING。因此，**无论我们如何旋转原始STRING，结果字符串必然是SS的子字符串**。因此，我们可以有效地在SS中找到子字符串并将其提取出来。

例如，如果我们以n = 2为单位正向旋转STRING，则结果为SS.subString(5,12)：

```text
Index  0   1   2   3   4   5   6 | 7   8   9   10  11  12  13
 SS    a   b   c   d   e   f   g | a   b   c   d   e   f   g
                                 |
Result a   b   c   d   e  [f   g   a   b   c   d   e]  f   g
```

现在，问题变成了识别自拼接字符串SS中预期的起始和结束索引，这个任务对我们来说相对简单：

- 起始索引：STRING.length – n
- 结束索引：StartIndex + STRING.length = 2 × STRING.length – n

接下来，让我们将这个想法“翻译”成Java代码：

```java
String rotateString2(String s, int c, boolean forward) {
    if (c < 0) {
        throw new IllegalArgumentException("Rotation character count cannot be negative!");
    }
    int len = s.length();
    int n = c % len;
    if (n == 0) {
        return s;
    }
    String ss = s + s;

    n = forward ? n : len - n;
    return ss.substring(len - n, 2 * len - n);
}
```

该方法也通过了我们的测试：

```java
// forward
assertEquals(EXPECT_1X, rotateString2(STRING, 1, true));
assertEquals(EXPECT_2X, rotateString2(STRING, 2, true));
assertEquals(EXPECT_3X, rotateString2(STRING, 3, true));
assertEquals(EXPECT_6X, rotateString2(STRING, 6, true));
assertEquals(EXPECT_7X, rotateString2(STRING, 7, true));
assertEquals(EXPECT_24X, rotateString2(STRING, 24, true));
                                                             
// backward
assertEquals(B_EXPECT_1X, rotateString2(STRING, 1, false));
assertEquals(B_EXPECT_2X, rotateString2(STRING, 2, false));
assertEquals(B_EXPECT_3X, rotateString2(STRING, 3, false));
assertEquals(B_EXPECT_6X, rotateString2(STRING, 6, false));
assertEquals(B_EXPECT_7X, rotateString2(STRING, 7, false));
assertEquals(B_EXPECT_24X, rotateString2(STRING, 24, false));
```

因此，它解决了我们的字符串旋转问题。

我们已经知道STRING的旋转结果将是SS的子字符串，值得注意的是，**我们可以使用此规则来检查一个字符串是否由另一个字符串旋转而来**：

```java
boolean rotatedFrom(String rotated, String rotateFrom) {
    return rotateFrom.length() == rotated.length() && (rotateFrom + rotateFrom).contains(rotated);
}
```

最后，我们来快速测试一下这个方法：

```java
assertTrue(rotatedFrom(EXPECT_7X, STRING));
assertTrue(rotatedFrom(B_EXPECT_3X, STRING));
assertFalse(rotatedFrom("abcefgd", STRING));
```

## 5. 总结

在本文中，我们首先分析了将字符串旋转n个字符的问题。然后，我们探讨了两种不同的解决方法。