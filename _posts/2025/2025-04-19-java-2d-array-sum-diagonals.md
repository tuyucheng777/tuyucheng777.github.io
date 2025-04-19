---
layout: post
title:  计算二维Java数组对角线值之和
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在Java中，处理二维数组(2D数组)非常常见，尤其是在涉及矩阵运算的任务中，其中一项任务就是计算二维数组中对角线值的总和。

在本教程中，我们将探索对二维数组中主对角线和次对角线的值求和的不同方法。

## 2. 问题介绍

首先，我们来快速了解一下问题。

二维数组构成一个矩阵，由于我们需要对对角线上的元素求和，**我们假设矩阵是n x n**，例如，一个4 x 4的二维数组：

```java
static final int[][] MATRIX = new int[][] {
    {  1,  2,  3,  4 },
    {  5,  6,  7,  8 },
    {  9, 10, 11, 12 },
    { 13, 14, 15, 100 }
};
```

接下来，让我们明确一下主对角线和次对角线的含义：

- 主对角线：**对角线从矩阵的左上角延伸到右下角**；例如，在上面的例子中，主对角线上的元素是1、6、11和100
- 次对角线：**对角线从右上角延伸至左下角**，在同一个例子中，4、7、10和13属于次对角线。

两个对角线值的总和如下：

```java
static final int SUM_MAIN_DIAGONAL = 118; // 1+6+11+100
static final int SUM_SECONDARY_DIAGONAL = 34; // 4+7+10+13
```

由于我们想要创建方法来覆盖两种对角线类型，因此让我们为它们创建一个[枚举](https://www.baeldung.com/a-guide-to-java-enums)：

```java
enum DiagonalType {
    Main, Secondary
}
```

稍后，**我们可以将DiagonalType传递给我们的解决方案来获得相应的结果**。

## 3. 识别对角线上的元素

要计算对角线值的和，**我们必须首先确定对角线上的元素**。在主对角线上，这非常简单，**当元素的行索引(rowIdx)和列索引(colIdx)相等时，该元素位于主对角线上**，例如MATRIX[0\][0\] = 1、MATRIX[1\][1\] = 6和MATRIX[3\][3\] = 100。

另一方面，**给定一个n x n矩阵，如果元素位于次对角线上，则rowIdx + colIdx = n – 1**。例如，在我们的4 x 4矩阵示例中，MATRIX[0\][3\] = 4(0 + 3 = 4 - 1)，MATRIX[1\][2\] = 7(1 + 2 = 4 - 1)，MATRIX[3\][0\] = 13(3 + 0 = 4 - 1)。因此，**colIdx = n – rowIdx – 1**。

现在我们了解了对角线元素的规则，让我们创建计算总和的方法。

## 4. 循环方法

**一种直接的方法是循环遍历行索引，根据所需的对角线类型对元素求和**：

```java
int diagonalSumBySingleLoop(int[][] matrix, DiagonalType diagonalType) {
    int sum = 0;
    int n = matrix.length;
    for (int rowIdx = 0; rowIdx < n; row++) {
        int colIdx = diagonalType == Main ? rowIdx : n - rowIdx - 1;
        sum += matrix[rowIdx][colIdx];
    }
    return sum;
}
```

正如我们在上面的实现中看到的，我们根据给定的diagonalType计算所需的colIdx，然后将rowIdx和colIdx上的元素相加到sum变量中。

接下来我们来测试一下这个解决方案是否能达到预期的效果：

```java
assertEquals(SUM_MAIN_DIAGONAL, diagonalSumBySingleLoop(MATRIX, Main));
assertEquals(SUM_SECONDARY_DIAGONAL, diagonalSumBySingleLoop(MATRIX, Secondary));
```

事实证明，这种方法可以对两种对角线类型求和正确的值。

## 5. 使用IntBinaryOperator对象的DiagonalType

基于循环的解决方案很简单，但是，在每个循环步骤中，我们必须检查diagonalType实例来确定colIdx，尽管diagonalType是一个在循环期间不会改变的参数。

接下来我们看看是否可以稍微改进一下。

**一个想法是为每个DiagonalType实例分配一个[IntBinaryOperator](https://www.baeldung.com/java-8-functional-interfaces#Operators)对象，这样我们就可以计算colIdx而无需检查我们拥有哪种对角线类型**：

```java
enum DiagonalType {
    Main((rowIdx, len) -> rowIdx),
    Secondary((rowIdx, len) -> (len - rowIdx - 1));

    public final IntBinaryOperator colIdxOp;

    DiagonalType(IntBinaryOperator colIdxOp) {
        this.colIdxOp = colIdxOp;
    }
}
```

如上代码所示，我们**为DiagonalType枚举添加了一个[IntBinaryOperator](https://www.baeldung.com/java-enum-values#adding-constructor)属性**，IntBinaryOperation是一个[函数式接口](https://www.baeldung.com/java-8-functional-interfaces)，它接收两个int参数并返回一个int值。在此示例中，我们使用两个[Lambda表达式](https://www.baeldung.com/java-8-Lambda-expressions-tips)作为枚举实例的IntBinaryOperator对象。

现在，我们可以删除for循环中的对角线类型检查的[三元运算](https://www.baeldung.com/java-ternary-operator)：

```java
int diagonalSumFunctional(int[][] matrix, DiagonalType diagonalType) {
    int sum = 0;
    int n = matrix.length;
    for (int rowIdx = 0; rowIdx < n; row++) {
        sum += matrix[rowIdx][diagonalType.colIdxOp.applyAsInt(rowIdx, n)];
    }
    return sum;
}
```

可以看到，**我们可以通过调用applyAsInt()直接调用diagonalType的colIdxOp函数来获取所需的colIdx**。 

当然，测试仍然通过：

```java
assertEquals(SUM_MAIN_DIAGONAL, diagonalSumFunctional(MATRIX, Main));
assertEquals(SUM_SECONDARY_DIAGONAL, diagonalSumFunctional(MATRIX, Secondary));
```

## 6. 使用Stream API

除了函数式接口，Java 8带来的另一个重要特性是[Stream API](https://www.baeldung.com/java-8-streams)。接下来，让我们使用Java 8的这两个特性来解决这个问题：

```java
public int diagonalSumFunctionalByStream(int[][] matrix, DiagonalType diagonalType) {
    int n = matrix.length;
    return IntStream.range(0, n)
        .map(i -> MATRIX[i][diagonalType.colIdxOp.applyAsInt(i, n)])
        .sum();
}
```

在此示例中，**我们用[IntStream.range()](https://www.baeldung.com/java-stream-of-and-intstream-range)替换了for循环。此外，[map()](https://www.baeldung.com/java-difference-map-and-flatmap)负责将每个索引(i)转换为对角线上所需的元素**，然后，[sum()](https://www.baeldung.com/java-stream-sum#using-intstreamsum)生成结果。

最后，这个解决方案也通过了测试：

```java
assertEquals(SUM_MAIN_DIAGONAL, diagonalSumFunctionalByStream(MATRIX, Main));
assertEquals(SUM_SECONDARY_DIAGONAL, diagonalSumFunctionalByStream(MATRIX, Secondary));
```

与最初的基于循环的解决方案相比，这种方法更流式且更易于阅读。

## 7. 总结

在本文中，我们探讨了计算二维Java数组中对角线值之和的不同方法，理解主对角线和次对角线的索引是解决问题的关键。