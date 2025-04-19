---
layout: post
title:  将二维数组转换为一维数组
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

[数组](https://www.baeldung.com/java-arrays-guide)是任何语言中最基本的数据结构，**虽然我们大多数情况下不会直接使用它们，但了解如何有效地操作它们可以显著提升我们的代码质量**。

在本教程中，我们将学习如何将二维数组转换为一维数组，通常称为[展平](https://www.baeldung.com/java-flatten-nested-collections)。例如，我们将{{1,2,3}, {4,5,6}, {7,8,9}}转换为{1,2,3,4,5,6,7,8,9}。

虽然我们将使用二维数组，但本教程中概述的思想可以应用于[任何维度](https://www.baeldung.com/java-jagged-arrays)的数组。在本文中，我们将使用[原始类型](https://www.baeldung.com/java-primitives)整数数组作为示例，但这些想法可以应用于任何数组。

## 2. 循环和原始类型数组

解决这个问题最简单的方法是使用[for](https://www.baeldung.com/java-for-loop)循环，我们可以用它将元素从一个数组传输到另一个数组。但是，为了提高性能，我们必须确定创建目标数组所需的元素总数。

如果所有数组的元素数量相同，那么这项任务就很简单了。在这种情况下，我们可以使用简单的数学运算来进行计算。**但是，如果使用[交错数组](https://en.wikipedia.org/wiki/Jagged_array#:~:text=In%20computer%20science%2C%20a%20jagged,edges%20when%20visualized%20as%20output.)，则需要分别遍历每个数组**：

```java
@ParameterizedTest
@MethodSource("arrayProvider")
void giveTwoDimensionalArray_whenFlatWithForLoopAndTotalNumberOfElements_thenGetCorrectResult(int [][] initialArray, int[] expected) {
    int totalNumberOfElements = 0;
    for (int[] numbers : initialArray) {
        totalNumberOfElements += numbers.length;
    }
    int[] actual = new int[totalNumberOfElements];
    int position = 0;
    for (int[] numbers : initialArray) {
        for (int number : numbers) {
            actual[position] = number;
            ++position;
        }
    }
    assertThat(actual).isEqualTo(expected);
}
```

另外，我们可以做一些改进，在第二个for循环中使用[System.arrayCopy()](https://www.baeldung.com/java-array-copy#the-system-class)：

```java
@ParameterizedTest
@MethodSource("arrayProvider")
void giveTwoDimensionalArray_whenFlatWithArrayCopyAndTotalNumberOfElements_thenGetCorrectResult(int [][] initialArray, int[] expected) {
    int totalNumberOfElements = 0;
    for (int[] numbers : initialArray) {
        totalNumberOfElements += numbers.length;
    }
    int[] actual = new int[totalNumberOfElements];
    int position = 0;
    for (int[] numbers : initialArray) {
        System.arraycopy(numbers, 0, actual,  position, numbers.length);
        position += numbers.length;
    }
    assertThat(actual).isEqualTo(expected);
}
```

**System.arrayCopy()相对[较快](https://www.baeldung.com/java-system-arraycopy-arrays-copyof-performance)，并且是推荐的数组复制方法，与[clone()](https://www.baeldung.com/java-copy-constructor#clone)方法一起使用**。但是，我们需要谨慎使用这些方法复制引用类型的数组，因为它们执行的是[浅拷贝](https://www.baeldung.com/java-deep-copy)。

从技术上讲，我们可以避免在第一次循环中计算元素的数量，并在必要时展开数组：

```java
@ParameterizedTest
@MethodSource("arrayProvider")
void giveTwoDimensionalArray_whenFlatWithArrayCopy_thenGetCorrectResult(int [][] initialArray, int[] expected) {
    int[] actual = new int[]{};
    int position = 0;
    for (int[] numbers : initialArray) {
        if (actual.length < position + numbers.length) {
            int[] newArray = new int[actual.length + numbers.length];
            System.arraycopy(actual, 0, newArray, 0, actual.length );
            actual = newArray;
        }
        System.arraycopy(numbers, 0, actual,  position, numbers.length);
        position += numbers.length;
    }
    assertThat(actual).isEqualTo(expected);
}
```

**然而，这种方法会降低性能，并将初始[时间复杂度](https://www.baeldung.com/cs/big-oh-asymptotic-complexity)从O(n)变为O(n^2)**。因此，应该避免使用这种方法，或者我们需要使用更[优化的算法](https://www.baeldung.com/cs/amortized-analysis)来增加数组大小，类似于List的实现。

## 3. 列表

关于列表，[Java Collection API](https://www.baeldung.com/java-collections)提供了一种更便捷的方式来管理元素集合。因此，如果我们使用[List](https://www.baeldung.com/java-arraylist)作为扁平化逻辑的返回类型，或者至少将其作为中间值持有者，我们可以简化代码：

```java
@ParameterizedTest
@MethodSource("arrayProvider")
void giveTwoDimensionalArray_whenFlatWithForLoopAndAdditionalList_thenGetCorrectResult(int [][] initialArray, int[] intArray) {
    List<Integer> expected = Arrays.stream(intArray).boxed().collect(Collectors.toList());
    List<Integer> actual = new ArrayList<>();
    for (int[] numbers : initialArray) {
        for (int number : numbers) {
            actual.add(number);
        }
    }
    assertThat(actual).isEqualTo(expected);
}
```

在这种情况下，我们不需要处理目标数组的扩展，List会透明地处理它。我们还可以将数组从第二维转换为List，以利用[addAll()](https://www.baeldung.com/java-add-items-array-list)方法：

```java
@ParameterizedTest
@MethodSource("arrayProvider")
void giveTwoDimensionalArray_whenFlatWithForLoopAndLists_thenGetCorrectResult(int [][] initialArray, int[] intArray) {
    List<Integer> expected = Arrays.stream(intArray).boxed().collect(Collectors.toList());
    List<Integer> actual = new ArrayList<>();
    for (int[] numbers : initialArray) {
        List<Integer> listOfNumbers = Arrays.stream(numbers).boxed().collect(Collectors.toList());
        actual.addAll(listOfNumbers);
    }
    assertThat(actual).isEqualTo(expected);
}
```

**由于我们不能在Collections中使用原始类型，因此[装箱](https://www.baeldung.com/cs/boxing-unboxing)会产生很大的开销，因此应谨慎使用**。当数组中的元素数量较多或性能至关重要时，最好避免使用[包装类](https://www.baeldung.com/java-wrapper-classes)。

## 4. Stream API

由于这类问题比较常见，[Stream API](https://www.baeldung.com/java-streams)提供了一种更方便的扁平化方法：

```java
@ParameterizedTest
@MethodSource("arrayProvider")
void giveTwoDimensionalArray_whenFlatWithStream_thenGetCorrectResult(int [][] initialArray, int[] expected) {
    int[] actual = Arrays.stream(initialArray).flatMapToInt(Arrays::stream).toArray();
    assertThat(actual).containsExactly(expected);
}
```

我们之所以使用flatMapToInt()方法，仅仅是因为我们处理的是原始类型数组。**引用类型的解决方案类似，但我们应该使用[flatMap()](https://www.baeldung.com/java-difference-map-and-flatmap)方法**。这是最直接、最易读的选项。不过，需要对Stream API有所了解。

## 5. 总结

System类、Collection和Stream API提供了许多与数组交互的便捷方法，但是，我们应该始终考虑这些方法的缺点，并根据具体情况选择最合适的方法。