---
layout: post
title:  Java中的基数排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在本教程中，我们将学习[基数排序](https://www.baeldung.com/cs/radix-sort)，分析其性能，并了解其实现。

**这里我们重点介绍如何使用基数排序对整数进行排序，但它不仅限于数字**，我们也可以用它来对其他类型进行排序，例如字符串。

为了简单起见，我们将重点关注以10为基数的十进制系统。

## 2. 算法概述

基数排序是一种根据数字位置对数字进行排序的排序算法，基本上，它使用数字中数字的位值。**与大多数其他排序算法(例如[归并排序](https://www.baeldung.com/java-merge-sort)、[插入排序](https://www.baeldung.com/java-insertion-sort)、[冒泡排序](https://www.baeldung.com/java-bubble-sort)不同)，它不比较数字**。

基数排序使用[稳定的排序算法](https://www.baeldung.com/stable-sorting-algorithms)作为子程序对数字进行排序，我们在这里使用了计数排序的变体作为子程序，该子程序使用基数对每个位置的数字进行排序。[计数排序](https://www.baeldung.com/java-counting-sort)是一种稳定的排序算法，在实践中效果很好。

基数排序的工作原理是从最低有效位(LSD)到最高有效位(MSD)的顺序对数字进行排序，我们也可以实现基数排序来处理从MSD开始的数字。

## 3. 一个简单的例子

让我们通过一个例子来看一下它是如何工作的，考虑以下数组：

![](/assets/images/2025/algorithms/javaradixsort01.png)

### 3.1 迭代1

我们将通过处理来自LSD的数字并移向MSD来对该数组进行排序。

因此让我们从个位上的数字开始：

![](/assets/images/2025/algorithms/javaradixsort02.png)

第一次迭代后，数组现在如下所示：

![](/assets/images/2025/algorithms/javaradixsort03.png)

请注意，数字已根据个位数字排序。

### 3.2 迭代2

我们继续讨论十位数字：

![](/assets/images/2025/algorithms/javaradixsort04.png)

现在数组看起来像这样：

![](/assets/images/2025/algorithms/javaradixsort05.png)

我们看到数字7占据了数组中的第一位，因为它的十位没有任何数字，或者我们也可以认为十位有一个0。

### 3.3 迭代3

我们继续讨论百位上的数字：

![](/assets/images/2025/algorithms/javaradixsort06.png)

经过这次迭代后，数组如下所示：

![](/assets/images/2025/algorithms/javaradixsort07.png)

算法到此停止，所有元素都已排序。

## 4. 实现

现在让我们看一下具体实现：
```java
void sort(int[] numbers) {
    int maximumNumber = findMaximumNumberIn(numbers);
    int numberOfDigits = calculateNumberOfDigitsIn(maximumNumber);
    int placeValue = 1;
    while (numberOfDigits-- > 0) {
        applyCountingSortOn(numbers, placeValue);
        placeValue *= 10;
    }
}
```

该算法的工作原理是找出数组中的最大值，然后计算其长度，此步骤可帮助我们确保为每个位值执行子程序。

例如，在数组[7,37,68,123,134,221,387,468,769\]中，最大数字为769，其长度为3。

因此，我们迭代并对每个位置的数字应用子程序3次：
```java
void applyCountingSortOn(int[] numbers, int placeValue) {
    int range = 10; // decimal system, numbers from 0-9

    // ...

    // calculate the frequency of digits
    for (int i = 0; i < length; i++) {
        int digit = (numbers[i] / placeValue) % range;
        frequency[digit]++;
    }

    for (int i = 1; i < range; i++) {
        frequency[i] += frequency[i - 1];
    }

    for (int i = length - 1; i >= 0; i--) {
        int digit = (numbers[i] / placeValue) % range;
        sortedValues[frequency[digit] - 1] = numbers[i];
        frequency[digit]--;
    }

    System.arraycopy(result, 0, numbers, 0, length);
}
```

在子程序中，我们使用基数(范围)来计算每个数字的出现次数并增加其频率。因此，0到9范围内的每个桶都会根据数字的频率获得相应的值，然后我们使用频率来定位数组中的每个元素，这也有助于我们最大限度地减少对数组进行排序所需的空间。

现在让我们测试该方法：
```java
@Test
void givenUnsortedArray_whenRadixSort_thenArraySorted() {
    int[] numbers = {387, 468, 134, 123, 68, 221, 769, 37, 7};
    RadixSort.sort(numbers);
    int[] numbersSorted = {7, 37, 68, 123, 134, 221, 387, 468, 769};
    assertArrayEquals(numbersSorted, numbers); 
}
```

## 5. 基数排序与计数排序

在子程序中，frequency数组的长度为10(0 - 9)。在计数排序的情况下，我们不使用range，frequency数组的长度将是数组中的最大数 + 1。因此，我们不将它们分成桶，而基数排序使用桶进行排序。

当数组的长度不比数组中的最大值小太多时，计数排序非常有效，而基数排序允许数组中存在更大的值。

## 6. 复杂性

基数排序的性能取决于选择对数字进行排序的稳定排序算法。

这里我们使用基数排序对以b为基数的n个数字数组进行排序，在我们的例子中，基数是10。我们应用了计数排序d次，其中d代表数字的位数，因此基数排序的时间复杂度变为O(d * (n + b))。

由于我们在这里使用了计数排序的变体作为子程序，因此空间复杂度为O(n + b)。

## 7. 总结

在本文中，我们描述了基数排序算法并说明了如何实现它。