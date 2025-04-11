---
layout: post
title:  在Java中旋转数组
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在本教程中，我们将学习一些Java中的[数组](https://www.baeldung.com/java-arrays-guide)旋转算法。

我们将了解如何将数组元素向右旋转k次，我们还将了解如何就地修改数组，尽管我们可能会使用额外的空间来计算旋转。

## 2. 数组旋转简介

在深入研究一些解决方案之前，让我们先讨论一下如何开始思考算法。

我们将用字母k作为旋转次数的别名。

### 2.1 寻找最小旋转

**我们将k转换为k除以数组长度的余数，或者k对数组长度取模(%)**，这使我们能够将较大的旋转数转换为相对较小的旋转数。

### 2.2 单元测试

**我们可能需要测试k是否小于、等于和大于数组长度**。例如，如果我们将一个包含6个元素的数组旋转8次，则只需要2次循环旋转。

类似地，如果k等于数组长度，我们不应该修改数组。因此，这是我们需要考虑的极端情况。

最后，我们还应该检查输入数组是否不为空或k是否大于0。

### 2.3 数组和旋转测试变量

我们将设置以下变量：

- arr作为要测试长度为6的数组
- rotationLtArrayLength = 1表示旋转小于数组长度
- rotationGtArrayLength = 8表示旋转大于数组长度
- ltArrayLengthRotation作为rotationLtArrayLength的解决方案
- gtArrayLengthRotation作为rotationGtArrayLength的解决方案

我们来看看它们的初始值：
```java
int[] arr = { 1, 2, 3, 4, 5, 6 };
int rotationLtArrayLength = 1;
int rotationGtArrayLength = arr.length + 2;
int[] ltArrayLengthRotation = { 6, 1, 2, 3, 4, 5 };
int[] gtArrayLengthRotation = { 5, 6, 1, 2, 3, 4 };
```

### 2.4 空间和时间复杂度

尽管如此，我们必须熟悉[时间和空间复杂性](https://www.baeldung.com/cs/time-vs-space-complexity)概念才能理解算法解决方案。

## 3. 暴力破解

尝试用[暴力破解](https://en.wikipedia.org/wiki/Brute-force_search)解决问题是一种常见的方法，这可能不是一种有效的解决方案。但是，它有助于理解问题空间。

### 3.1 算法

**我们通过k步旋转来解决，同时每一步将元素移动一个单位**。

我们来看看这个方法：
```java
void bruteForce(int[] arr, int k) {
    // check invalid input
    k %= arr.length;
    int temp;
    int previous;
    for (int i = 0; i < k; i++) {
        previous = arr[arr.length - 1];
        for (int j = 0; j < arr.length; j++) {
            temp = arr[j];
            arr[j] = previous;
            previous = temp;
        }
    }
}
```

### 3.2 代码洞察

我们用嵌套循环对数组长度进行两次迭代，首先，我们在索引i处获取要旋转的值：
```java
for (int i = 0; i < k; i++) {
    previous = arr[arr.length - 1];
    // nested loop
}
```

然后，我们使用临时变量将嵌套[for](https://www.baeldung.com/java-for-loop)循环中的所有元素移动一格。
```java
for (int j = 0; j < arr.length; j++) {
    temp = arr[j];
    arr[j] = previous;
    previous = temp;
}
```

### 3.3 复杂度分析

- **时间复杂度：O(n×k)**，所有数字移动一步O(n)，共k次
- **空间复杂度：O(1)**，使用常数额外空间

### 3.4 单元测试

让我们编写一个当k小于数组长度时的暴力算法测试：
```java
@Test
void givenInputArray_whenUseBruteForceRotationLtArrayLength_thenRotateArrayOk() {
    bruteForce(arr, rotationLtArrayLength);
    assertArrayEquals(ltArrayLengthRotation, arr);
}
```

同样，我们测试k是否大于数组长度：
```java
@Test
void givenInputArray_whenUseBruteForceRotationGtArrayLength_thenRotateArrayOk() {
    bruteForce(arr, rotationGtArrayLength);
    assertArrayEquals(gtArrayLengthRotation, arr);
}
```

最后，让我们测试k是否等于数组长度：
```java
@Test
void givenInputArray_whenUseBruteForceRotationEqArrayLength_thenDoNothing() {
    int[] expected = arr.clone();
    bruteForce(arr, arr.length);
    assertArrayEquals(expected, arr);
}
```

## 4. 使用额外数组的旋转

**我们使用一个额外的数组将每个元素放置在正确的位置**，然后，我们将其复制回原始数组。

### 4.1 算法

为了获得旋转后的位置，我们需要找到数组长度的(i + k)位置模数(%)：
```java
void withExtraArray(int[] arr, int k) {
    // check invalid input

    int[] extraArray = new int[arr.length];
    for (int i = 0; i < arr.length; i++) {
        extraArray[(i + k) % arr.length] = arr[i];
    }
    System.arraycopy(extraArray, 0, arr, 0, arr.length);
}
```

值得注意的是，虽然不是必需的，但我们可以返回额外的数组。

### 4.2 代码洞察

我们将每个元素从i移动到额外空间数组中的(i + k) % arr.length位置。

最后，我们使用[System.arraycopy()](https://www.baeldung.com/java-array-copy#the-system-class)将其复制到原始数组。

### 4.3 复杂度分析

- **时间复杂度：O(n)**，我们先对新数组进行一次旋转，然后再将新数组复制到原始数组中
- **空间复杂度：O(n)**，我们使用另一个相同大小的数组来存储旋转值

### 4.4 单元测试

当k小于数组长度时，让我们用这个额外的数组算法来测试旋转：
```java
@Test
void givenInputArray_whenUseExtraArrayRotationLtArrayLength_thenRotateArrayOk() {
    withExtraArray(arr, rotationLtArrayLength);
    assertArrayEquals(ltArrayLengthRotation, arr);
}
```

同样，我们测试k是否大于数组长度：
```java
@Test
void givenInputArray_whenUseExtraArrayRotationGtArrayLength_thenRotateArrayOk() {
    withExtraArray(arr, rotationGtArrayLength);
    assertArrayEquals(gtArrayLengthRotation, arr);
}

```

最后，让我们测试k是否等于数组长度：
```java
@Test
void givenInputArray_whenUseExtraArrayWithRotationEqArrayLength_thenDoNothing() {
    int[] expected = arr.clone();
    withExtraArray(arr, arr.length);
    assertArrayEquals(expected, arr);
}
```

## 5. 循环替换

**我们可以每次将元素替换到所需位置**，但是，这会丢失原始值。因此，我们将它存储在一个临时变量中，然后，我们可以将旋转后的元素放置n次，其中n是数组长度。

### 5.1 算法

我们可以在k = 2的图中看到循环替换的逻辑：

![](/assets/images/2025/algorithms/javarotatearrays01.png)

因此，我们从索引0开始并进行2步循环。

但是，这种方法存在一个问题。我们可能会注意到，我们可以返回到初始数组位置，在这种情况下，我们将重新开始一个常规的for循环。

最后，我们将使用count变量跟踪替换的元素，当count等于数组长度时，循环完成。

我们来看看代码：
```java
void cyclicReplacement(int[] arr, int k) {
    // check invalid input

    k = k % arr.length;
    int count = 0;
    for (int start = 0; count < arr.length; start++) {
        int current = start;
        int prev = arr[start];
        do {
            int next = (current + k) % arr.length;
            int temp = arr[next];
            arr[next] = prev;
            prev = temp;
            current = next;
            count++;
        } while (start != current);
    }
}
```

### 5.2 代码洞察

对于这个算法，我们需要关注两个部分。

第一个是外循环，我们在k步循环中对替换的元素的每n / k部分进行迭代：
```java
for (int start = 0; count < arr.length; start++) {
    int current = start;
    int prev = arr[start];
    do {
        // do loop
    } while (start != current);
}
```

因此，我们使用[do-while循环](https://www.baeldung.com/java-do-while-loop)将值移动k步(同时保存替换的值)直到达到最初开始的数字并返回到外层循环：
```java
int next = (current + k) % arr.length;
int temp = arr[next];
arr[next] = prev;
prev = temp;
current = next;
count++;
```

### 5.3 复杂度分析

- **时间复杂度：O(n)**
- **空间复杂度：O(1)**，使用常数额外空间

### 5.4 单元测试

我们来测试一下当k小于数组长度时的循环替换数组算法：
```java
@Test
void givenInputArray_whenUseCyclicReplacementRotationLtArrayLength_thenRotateArrayOk() {
    cyclicReplacement(arr, rotationLtArrayLength);
    assertArrayEquals(ltArrayLengthRotation, arr);
}
```

同样，我们测试k是否大于数组长度：
```java
@Test
void givenInputArray_whenUseCyclicReplacementRotationGtArrayLength_thenRotateArrayOk() {
    cyclicReplacement(arr, rotationGtArrayLength);
    assertArrayEquals(gtArrayLengthRotation, arr);
}

```

最后，让我们测试k是否等于数组长度：
```java
@Test
void givenInputArray_whenUseCyclicReplacementRotationEqArrayLength_thenDoNothing() {
    int[] expected = arr.clone();
    cyclicReplacement(arr, arr.length);
    assertArrayEquals(expected, arr);
}
```

## 6. 反转

这是一个简单但并不平凡的算法，**当我们旋转时，我们可能会注意到数组后端的k个元素移到了前面，而前端的其余元素则向后移动**。

### 6.1 算法

我们通过3个逆向步骤来解决：

- 数组的所有元素
- 前k个元素
- 其余(nk)元素

我们来看看代码：
```java
void reverse(int[] arr, int k) {
    // check invalid input

    k %= arr.length;
    reverse(arr, 0, arr.length - 1);
    reverse(arr, 0, k - 1);
    reverse(arr, k, arr.length - 1);
}

void reverse(int[] nums, int start, int end) {
    while (start < end) {
        int temp = nums[start];
        nums[start] = nums[end];
        nums[end] = temp;
        start++;
        end--;
    }
}
```

### 6.2 代码洞察

我们使用了一种类似于暴力破解嵌套循环的辅助方法，不过，这次我们分3个不同的步骤进行：

我们反转整个数组：
```java
reverse(arr, 0, arr.length - 1);
```

然后，我们反转前k个元素。
```java
reverse(arr, 0, k - 1);
```

最后，我们反转剩余的元素：
```java
reverse(arr, k, arr.length - 1);
```

虽然这似乎引入了冗余，但它可以让我们在线性时间内完成任务。

### 6.3 复杂度分析

- **时间复杂度：O(n)**，n个元素一共被反转3次
- **空间复杂度：O(1)**，使用常数额外空间

### 6.4 单元测试

我们来测试一下当k小于数组长度时的反向算法：
```java
@Test
void givenInputArray_whenUseReverseRotationLtArrayLength_thenRotateArrayOk() {
    reverse(arr, rotationLtArrayLength);
    assertArrayEquals(ltArrayLengthRotation, arr);
}
```

同样，我们测试k是否大于数组长度：
```java
@Test
void givenInputArray_whenUseReverseRotationGtArrayLength_thenRotateArrayOk() {
    reverse(arr, rotationGtArrayLength);
    assertArrayEquals(gtArrayLengthRotation, arr);
}

```

最后，让我们测试k是否等于数组长度：
```java
@Test
void givenInputArray_whenUseReverseRotationEqArrayLength_thenDoNothing() {
    int[] expected = arr.clone();
    reverse(arr, arr.length);
    assertArrayEquals(expected, arr);
}
```

## 7. 总结

在本文中，我们了解了如何对数组进行k次旋转。我们从暴力算法开始，然后逐步深入到更复杂的算法，例如不占用额外空间的反向或循环替换，我们还讨论了时间和空间复杂性。最后，我们给出了单元测试和一些边缘情况。