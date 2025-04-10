---
layout: post
title:  使用Java中的for循环创建三角形
category: algorithms
copyright: algorithms
excerpt: 算法
---

## 1. 简介

在本教程中，我们将探索在Java中打印三角形的几种方法。

当然，三角形有很多种类型，这里我们只讨论其中的两种：**直角三角形和等腰三角形**。

## 2. 构建直角三角形

直角三角形是我们要研究的最简单的三角形类型，让我们快速看一下我们想要获得的输出：
```text
*
**
***
****
*****
```

这里，我们注意到三角形由5行组成，每行的星号数量等于当前行数。当然，这个观察结果可以推广：**对于从1到N的每一行，我们必须打印r个星号，其中r是当前行，N是总行数**。

因此，让我们使用两个for循环来构建三角形：
```java
public static String printARightTriangle(int N) {
    StringBuilder result = new StringBuilder();
    for (int r = 1; r <= N; r++) {
        for (int j = 1; j <= r; j++) {
            result.append("*");
        }
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

## 3. 构建等腰三角形

现在，我们来看看等腰三角形的形状：
```text
    *
   ***
  *****
 *******
*********
```

在这种情况下我们看到了什么？我们注意到，**除了星号之外，我们还需要为每行打印一些空格**。因此，我们必须计算出每行需要打印多少个空格和星号。当然，空格和星号的数量取决于当前行。

首先，我们看到第一行需要打印4个空格，而当我们沿着三角形向下时，需要打印3个空格、2个空格、1个空格，最后一行不需要任何空格。**概括起来，我们需要为每一行打印N – r个空格**。

其次，与第一个例子相比，我们意识到这里我们需要奇数个星号：1、3、5、7...

因此，**我们需要为每行打印r x 2– 1个星号**。

### 3.1 使用嵌套for循环

根据以上观察，让我们创建第二个示例：
```java
public static String printAnIsoscelesTriangle(int N) {
    StringBuilder result = new StringBuilder();
    for (int r = 1; r <= N; r++) {
        for (int sp = 1; sp <= N - r; sp++) {
            result.append(" ");
        }
        for (int c = 1; c <= (r * 2) - 1; c++) {
            result.append("*");
        }
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

### 3.2 使用单个for循环

实际上，我们还有另一种**仅包含单个for循环的方法-它使用[Apache Commons Lang3库](https://www.baeldung.com/java-commons-lang-3)**。

我们将使用for循环遍历三角形的行，就像在前面的示例中所做的那样，然后，我们将使用StringUtils.repeat()方法为每行生成必要的字符：
```java
public static String printAnIsoscelesTriangleUsingStringUtils(int N) {
    StringBuilder result = new StringBuilder();

    for (int r = 1; r <= N; r++) {
        result.append(StringUtils.repeat(' ', N - r));
        result.append(StringUtils.repeat('*', 2 * r - 1));
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

或者，我们可以用[substring()](https://www.baeldung.com/java-substring)[方法](https://www.baeldung.com/java-substring)做一个巧妙的技巧。

我们可以提取上面的StringUtils.repeat()方法来构建辅助字符串，然后对其应用String.substring()方法，**辅助字符串是打印三角形行所需的最大空格数和最大星号数的串联**。

查看前面的例子，我们注意到第一行最多需要N – 1个空格，最后一行最多需要N x 2 – 1个星星：
```java
String helperString = StringUtils.repeat(' ', N - 1) + StringUtils.repeat('*', N * 2 - 1);
// for N = 10, helperString = "    *********"
```

例如，当N=5且r=3时，我们需要打印“*****”，它包含在helperString变量中，我们需要做的就是找到substring()方法的正确公式。

现在，让我们看完整的例子：
```java
public static String printAnIsoscelesTriangleUsingSubstring(int N) {
    StringBuilder result = new StringBuilder();
    String helperString = StringUtils.repeat(' ', N - 1) + StringUtils.repeat('*', N * 2 - 1);

    for (int r = 0; r < N; r++) {
        result.append(helperString.substring(r, N + 2 * r));
        result.append(System.lineSeparator());
    }
    return result.toString();
}
```

类似地，只需稍微更改，我们就可以让三角形打印倒置。

## 4. 复杂度

回顾第一个例子，我们会注意到一个外循环和一个内循环，每个循环最多包含N步。**因此，时间复杂度为O(N^2)，其中N是三角形的行数**。

第二个例子类似-唯一的区别是我们有两个内循环，它们是连续的并且不会增加时间复杂度。

但是，第三个示例仅使用具有N个步骤的for循环。但是，在每一步中，我们都会在辅助字符串上调用StringUtils.repeat()方法或substring()方法，每个方法的复杂度均为O(N)。因此，总体时间复杂度保持不变。

最后，如果我们讨论辅助空间，可以很快意识到，对于所有示例，复杂度都保留在[StringBuilder](https://www.baeldung.com/java-string-builder-string-buffer)变量中，**通过将整个三角形添加到result变量，我们的复杂性不可能小于O(N^2)**。

当然，如果我们直接打印字符，前两个示例的空间复杂度将是恒定的。但是，第三个示例使用了辅助字符串，空间复杂度将是O(N)。

## 5. 总结

在本教程中，我们学习了如何在Java中打印两种常见类型的三角形。

首先，我们学习了直角三角形，这是我们可以在Java中打印的最简单的三角形类型。然后，我们探索了两种构建等腰三角形的方法；第一种方法仅使用for循环，另一种方法利用StringUtils.repeat()和String.substring()方法，帮助我们编写更少的代码。

最后，我们分析了每个示例的时间和空间复杂度。