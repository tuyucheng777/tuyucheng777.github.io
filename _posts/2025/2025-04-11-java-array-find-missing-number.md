---
layout: post
title:  在Java中从给定数组中查找缺失的数字
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在Java中，从[数组](https://www.baeldung.com/java-arrays-guide)中查找指定范围内的缺失数字在各种场景中都很有用，例如数据验证、确保完整性或识别数据集中的间隙。

在本教程中，**我们将学习多种方法，从整数范围[1-N\]的数组中查找单个缺失数字。此外，我们还将学习如何从数组中查找所有缺失的数字**。

## 2. 理解场景

假设我们有一个包含[1-9\]范围内整数的数字数组(包括1-9)：
```java
int[] numbers = new int[] { 1, 4, 5, 2, 7, 8, 6, 9 };
```

**现在，我们的目标是从[1-9\]范围内的数组中找到缺失的数字**。

为了概括问题陈述，我们可以计算数组的长度并设置上限N：
```java
int N = numbers.length + 1;
```

在接下来的部分中，我们将学习从范围[1-N\]内的给定数组中查找缺失数字的不同方法。

## 3. 使用算术和

让我们首先使用算术和从数字数组中找出缺失的数字。

首先，我们计算范围[1-N\]内的[等差数列](https://en.wikipedia.org/wiki/Arithmetic_progression)的预期和以及数组的实际和：
```java
int expectedSum = (N * (N + 1)) / 2;
int actualSum = Arrays.stream(numbers).sum();
```

接下来，我们可以从expectedSum中减去actualSum来得到missingNumber：
```java
int missingNumber = expectedSum - actualSum;
```

最后我们来验证一下结果：
```java
assertEquals(3, missingNumber);
```

## 4. 使用XOR属性

或者，我们可以使用[异或运算符](https://en.wikipedia.org/wiki/Exclusive_or)(^)的两个有趣属性来解决我们的用例：

- X^X = 0：当我们将一个数字与自身进行异或时，我们得到0。
- X^0 = X：当我们将一个数字与0进行异或时，我们得到相同的数字。

首先，我们将使用[Reduce](https://www.baeldung.com/java-stream-reduce)函数对封闭范围[1-9\]内的所有整数值执行异或运算：
```java
int xorValue = IntStream.rangeClosed(1, N).reduce(0, (a, b) -> a ^ b);
```

我们分别使用0和(a, b) -> a ^ b(一个[Lambda表达式](https://www.baeldung.com/java-lambdas-vs-anonymous-class#lambda-expression))作为reduce()操作的[恒等式和累加器](https://www.baeldung.com/java-stream-reduce#a-quick-intro-to-identifier-accumulator-combiner)。

接下来，我们将继续对数字数组中的整数值进行异或运算：
```java
xorValue = Arrays.stream(numbers).reduce(xorValue, (x, y) -> x ^ y);
```

**由于除缺失数字之外的每个数字都出现两次，因此xorValue将仅包含范围[1-9\]内的数字数组中的缺失数字**。

最后，我们应该验证我们的方法是否给出了正确的结果：
```java
assertEquals(3, xorValue);
```

## 5. 使用排序

我们的输入数组numbers预期包含[1-N\]范围内的所有连续值(除了缺失的数字)，因此，如果我们对数组进行排序，就可以方便地在没有连续数字的地方找到缺失的数字。

首先，我们对数字数组进行排序：
```java
Arrays.sort(numbers);
```

接下来，我们可以迭代数字数组并检查索引处的值是否为index + 1：
```java
int missingNumber = -1;
for (int index = 0; index < numbers.length; index++) {
    if (numbers[index] != index + 1) {
        missingNumber = index + 1;
        break;
    }
}
```

**当条件不满足时，意味着数组中缺少预期值index + 1**，因此，我们设置了missingNumber并提前退出循环。

最后，让我们检查一下是否得到了所需的输出：
```java
assertEquals(3, missingNumber);
```

结果看起来正确。但是，我们必须注意，在这种情况下，我们改变了原始输入数组。

## 6. 使用布尔数组进行跟踪

在排序方法中，有两个主要缺点：

- 排序的间接成本
- 原始输入数组的变异

我们可以通过使用布尔数组来跟踪当前数字来缓解这些问题。

首先，我们将present定义为大小为N的布尔数组：
```java
boolean[] present = new boolean[N];
```

我们必须记得将N初始化为numbers.length + 1。

接下来，**我们将迭代数字数组并标记present数组中每个数字的存在**：
```java
int missingNumber = -1;
Arrays.stream(numbers).forEach(number -> present[number - 1] = true);
```

此外，我们将执行另一次迭代，但在present数组上，查找未标记为存在的数字：
```java
for (int index = 0; index < present.length; index++) {
    if (!present[index]) {
        missingNumber = index + 1;
        break;
    }
}
```

最后，让我们通过检查missingNumber变量的值来验证我们的方法：
```java
assertEquals(3, missingNumber);
```

另外，需要注意的是，我们使用了N个字节的额外空间，因为每个布尔值在Java中将占用1个字节。

## 7. 使用Bitset进行跟踪

我们可以通过在布尔数组上使用[Bitset](https://www.baeldung.com/java-bitset)来优化空间复杂度。
```java
BitSet bitSet = new BitSet(N);
```

通过此初始化，**我们将仅使用足够的空间来表示N位**。当N的值相当高时，这是一个相当大的优化。

接下来，让我们迭代numbers数组并通过在bitset中设置一个位来标记它们的存在：
```java
for (int num : numbers) {
    bitSet.set(num);
}
```

现在，**我们可以通过检查未设置的位来找到丢失的数字**：
```java
int missingNumber = bitSet.nextClearBit(1);
```

最后，让我们确认missingNumber中的值是正确的：
```java
assertEquals(3, missingNumber);
```

## 8. 找出所有缺失的数字

为了找到多个缺失数字，我们可以扩展前面部分讨论的解决方案。例如，我们可以对BitSet跟踪方法进行一些修改，以处理多个缺失数字。

首先，让我们确定给定数组中的最大值，此最大值确定从1到N的范围的上限(N)：
```java
int[] numbersWithMultipleMissing = new int[] { 1, 5, 2, 8, 9 };
int N = Arrays.stream(numbersWithMultipleMissing)
    .max()
    .getAsInt();
```

接下来，让我们创建allBitSet，它保存从整数1到N的所有设置位：
```java
BitSet allBitSet = new BitSet(N + 1);
IntStream.rangeClosed(1, N)
    .forEach(allBitSet::set);
```

然后，我们可以创建presentBitSet，在其中，我们为numbersWithMultipleMissing数组中存在的每个数字设置位：
```java
BitSet presentBitSet = new BitSet(N + 1);
Arrays.stream(numbersWithMultipleMissing)
    .forEach(presentBitSet::set);
```

现在，我们可以在allBitSet和presentBitSet之间执行逻辑与运算，将allBitSet和presentBitSet中的公共位设置为true，同时将不常见的位保留为false。
```java
allBitSet.and(presentBitSet);
```

最后，让我们从1到N的范围进行迭代，并检查allBitSet中所有未设置的位，每个未设置的位对应于1到N范围内缺失的一个数字：
```java
List<Integer> result = IntStream.rangeClosed(1, N)
    .filter(i -> !allBitSet.get(i))
    .boxed()
    .sorted()
    .collect(Collectors.toList());
```

在上述逻辑中，我们按排序顺序收集result列表中所有缺失的数字，这确保以可预测的顺序将结果与预期输出进行比较：
```java
assertEquals(result, Arrays.asList(3, 4, 6, 7));
```

## 9. 总结

在本文中，我们学习了如何从数组中查找缺失的数字。此外，我们探索了多种解决用例的方法，例如算术和、异或运算、排序和其他数据结构，如Bitset和布尔数组。此外，我们还扩展了逻辑，以便从数组中查找多个缺失的数字。