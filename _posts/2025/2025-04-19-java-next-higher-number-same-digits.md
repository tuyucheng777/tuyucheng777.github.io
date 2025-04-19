---
layout: post
title:  找到下一个具有相同数字的更大数字
category: algorithms
copyright: algorithms
excerpt: 数字
---

## 1. 简介

在本教程中，我们将学习如何在Java中查找与原始数字具有相同数字集的下一个更大数字。这个问题可以通过使用排列、排序和双指针方法的概念来解决。

## 2. 问题陈述

给定一个正整数，我们需要找到下一个使用完全相同数字的更大数字。例如，如果输入是123，我们的目标是重新排列它的数字，以形成下一个具有相同数字的更大数字。在本例中，下一个更大的数字是132。

如果输入是654或444，那么我们返回-1来表示没有下一个更大的数字。

## 3. 使用排列

在这个方法中，我们将利用排列来查找下一个与输入数字相同且更大的数字。**我们将生成输入数字中所有可能的排列，并将它们添加到[TreeSet](https://www.baeldung.com/java-tree-set)中，以确保唯一性和有序性**。 

### 3.1 实现

**首先，我们实现一个方法findPermutations()来生成输入数字num中数字的所有排列，并将它们添加到TreeSet中**：

```java
void findPermutations(int num, int index, StringBuilder sb, Set<Integer> hs) {
    if (index == String.valueOf(num).length()) {
        hs.add(Integer.parseInt(sb.toString()));
        return;
    }
    //...
}
```

该方法首先检查当前索引是否等于输入数字的长度，如果是，则表示排列已完全生成，然后将该排列添加到TreeSet中并返回以结束递归。

**否则，我们从当前索引开始迭代数字的数字以生成所有排列**：

```java
for (int i = index; i < String.valueOf(num).length(); i++) {
    char temp = sb.charAt(index);
    sb.setCharAt(index, sb.charAt(i));
    sb.setCharAt(i, temp);
    // ...
}
```

在每次迭代中，我们将当前索引index处的字符与迭代索引i处的字符交换，**此交换操作实际上创建了不同的数字组合**。

交换之后，该方法使用更新后的索引和修改后的[StringBuilder](https://www.baeldung.com/java-string-builder-string-buffer)[递归](https://www.baeldung.com/java-recursion)调用自身：

```java
findPermutations(num, index + 1, sb, hs);
temp = sb.charAt(index);
sb.setCharAt(index, sb.charAt(i));
sb.setCharAt(i, temp); // Swap back after recursion
```

递归调用结束后，我们将字符交换回其原始位置，以便在后续迭代中保持sb的完整性。

让我们将逻辑封装在一个方法中：

```java
int findNextHighestNumberUsingPermutation(int num) {
    Set<Integer> hs = new TreeSet();
    StringBuilder sb = new StringBuilder(String.valueOf(num));
    findPermutations(num, 0, sb, hs);
    for (int n : hs) {
        if (n > num) {
            return n;
        }
    }
    return -1;
}
```

**所有排列生成后，我们会遍历TreeSet，找到大于输入数字的最小数字**。如果找到，则将其作为下一个更大的数字。否则，我们返回-1，表示不存在这样的数字。

### 3.2 测试

让我们验证一下排列解决方案：

```java
assertEquals(536497, findNextHighestNumberUsingPermutation(536479));
assertEquals(-1, findNextHighestNumberUsingPermutation(987654));
```

### 3.3 复杂度分析

**此实现的时间复杂度在最坏情况下为O(n!)，其中n是输入数字的位数**。findPermutations()函数会生成数字n!的所有可能排列，这仍然是时间复杂度的主要因素。

虽然TreeSet为插入和检索提供了对数复杂度(log n)，但它对整体时间复杂度并没有显著影响。

**在最坏的情况下，所有排列都是唯一的(没有重复)，TreeSet可能容纳所有n!个数字的排列**，这导致空间复杂度为O(n!)。

## 4. 使用排序

在这种方法中，我们将采用排序方法来确定与给定输入数字具有相同数字的下一个更大数字。

### 4.1 实现

我们首先定义一个名为findNextHighestNumberUsingSorting()的方法：

```java
int findNextHighestNumberUsingSorting(int num) {
    String numStr = String.valueOf(num);
    char[] numChars = numStr.toCharArray();
    int pivotIndex;
    // ...
}
```

**在该方法内部，我们将输入的数字转换为字符串，然后再转换为字符数组**，我们还初始化一个变量pivotIndex来标识枢轴点。

接下来，我们从最右边的数字到左边迭代numChars数组，以确定小于或等于其右边邻居的最大数字：

```java
for (pivotIndex = numChars.length - 1; pivotIndex > 0; pivotIndex--) {
    if (numChars[pivotIndex] > numChars[pivotIndex - 1]) {
        break;
    }
}
```

这个数字将成为枢轴点，用于标识数字降序排列中的断点。**如果此条件成立，则意味着我们找到了一个比其左侧相邻数字更大的数字(潜在枢轴)**。然后我们跳出循环，因为我们不需要进一步搜索更大的降序数字。

如果没有找到这样的枢轴，则意味着该数字已经按降序排列，因此我们返回-1：

```java
if (pivotIndex == 0) {
    return -1;
}
```

确定pivot后，代码会搜索pivot右侧仍然大于pivot本身的最小数字：

```java
int pivot = numChars[pivotIndex - 1];
int minIndex = pivotIndex;

for (int j = pivotIndex + 1; j < numChars.length; j++) {
    if (numChars[j] > pivot && numChars[j] < numChars[minIndex]) {
        minIndex = j;
    }
}
```

**这个数字稍后会与pivot值交换，得到下一个更大的数字**。我们从pivot值之后的位置开始，使用循环遍历数组，找到大于pivot值的最小数字。

一旦找到大于枢轴的最小数字，我们就将其位置与枢轴交换：

```java
swap(numChars, pivotIndex - 1, minIndex);
```

**这种交换本质上是将大于枢轴的最小数字带到枢轴之前的位置**。

为了创建下一个按字典顺序排列的更大的数字，代码需要按升序对枢轴右侧的数字进行排序：

```java
Arrays.sort(numChars, pivotIndex, numChars.length);
return Integer.parseInt(new String(numChars));
```

这会产生大于原始数字的最小数字排列。

### 4.2 测试

现在，让我们验证我们的排序实现：

```java
assertEquals(536497, findNextHighestNumberUsingSorting(536479));
assertEquals(-1, findNextHighestNumberUsingSorting(987654));
```

在第一个测试用例中，pivot位于索引4处(数字7)。接下来，为了找到大于主元(7)的最小数字，我们从pivotIndex + 1迭代到末尾。9大于pivot(7)，因此大于pivot的最小数字位于索引5处(数字9)。

现在，我们将numChars[4\](7)与numChars[5\](9)交换，交换和排序后，numChars数组变为[5,3,6,4,9,7\]。

### 4.3 复杂度分析

**此实现的时间复杂度为O(n log n)，空间复杂度为O(n)**，这是因为对数字进行排序的实现时间为O(n log n)，并且使用字符数组来存储数字。

## 5. 使用双指针

这种方法比排序更有效率，它利用两个指针来找到所需的数字，并对其进行操作以获得下一个更大的数字。

### 5.1 实现

在开始主要逻辑之前，我们创建两个辅助方法来简化数组内字符的操作：

```java
void swap(char[] numChars, int i, int j) {
    char temp = numChars[i];
    numChars[i] = numChars[j];
    numChars[j] = temp;
}

void reverse(char[] numChars, int i, int j) {
    while (i < j) {
        swap(numChars, i, j);
        i++;
        j--;
    }
}
```

然后我们首先定义一个名为findNextHighestNumberUsingTwoPointer()的方法：

```java
int findNextHighestNumberUsingTwoPointer (int num) {
    String numStr = String.valueOf(num);
    char[] numChars = numStr.toCharArray();
    int pivotIndex = numChars.length - 2;
    int minIndex = numChars.length - 1;
    // ...
}
```

在方法内部，我们将输入的数字转换为字符数组并初始化两个变量：

- pivotIndex：从数组右侧跟踪枢轴索引
- minIndex：查找大于枢轴的数字

我们将pivotIndex初始化为倒数第二位数字，因为如果从最后一位数字(即numChars.length - 1)开始，其右侧就没有数字可以比较。**随后，我们使用while循环从右侧找到第一个索引pivotIndex，其数字小于或等于其右侧的数字**：

```java
while (pivotIndex >= 0 && numChars[pivotIndex] >= numChars[pivotIndex+1]) {
    pivotIndexi--;
}
```

如果当前数字较小，则表示我们找到了上升趋势突破的枢轴点，然后循环将终止。

如果没有找到这样的索引，则意味着输入数字是最大的可能排列，并且没有更大的可能排列：

```java
if (pivotIndex == -1) {
    result = -1;
    return;
}
```

否则，我们使用另一个while循环来查找从右边开始第一个索引minIndex，其数字大于枢轴数字pivotIndex：

```java
while (numChars[minIndex] <= numChars[pivotIndex]) {
    minIndex--;
}
```

接下来，我们使用swap()函数交换索引pivotIndex和minIndex处的数字：

```java
swap(numChars, pivotIndex, minIndex);
```

这种交换操作确保我们在生成排列时探索不同的数字组合。

**最后，我们不进行排序，而是反转索引pivotIndex右侧的子字符串，以获得大于原始数字的最小排列**：

```java
reverse(numChars, pivotIndex+1, numChars.length-1);
return Integer.parseInt(new String(numChars));
```

这实际上使用剩余的数字创建了大于原始数字的最小可能数字。

### 5.2 测试

让我们验证两个指针的实现：

```java
assertEquals(536497, findNextHighestNumberUsingSorting(536479));
assertEquals(-1, findNextHighestNumberUsingSorting(987654));
```

在第一个测试用例中，pivotIndex保持为其初始化值(4)，因为numChars[4\](7)不大于numChars[5\](9)，因此，while循环立即中断。

在第二个while循环中，由于numChars[5\](9)不小于numChars[4\](7)，因此minIndex仍为其初始值(5)，while循环再次终止。

接下来，我们将numChars[4\](7)与numChars[5\](9)交换。由于pivotIndex + 1为5且minIndex也为5，因此无需反转，numChars数组变为[5,3,6,4,9,7\]。

### 5.3 复杂度分析

**在最坏的情况下，遍历数字的循环可能会遍历整个字符数组numChars两次**。当输入数字已经按降序排列时(例如987654)，就会发生这种情况。

**因此，该函数的时间复杂度为O(n)。此外，由于它为指针和临时变量使用了恒定数量的额外空间，因此空间复杂度为O(1)**。

## 6. 比较

下表总结了三种方法的比较和推荐的用例：

| 方法  |  时间复杂度| 空间复杂度|      用例       |
|:---:| :----------: | :--------: |:-------------:|
| 排列  |    O(n!)    |  O(n!)   | 当输入数字位数较少时使用  |
| 排序  | O(n log n) |    	O(n)    | 当实现的简单性很重要时使用 |
| 双指针 |     在)     |O(1)    | 当输入数字位数较多时使用  |

## 7. 总结

在本文中，我们探讨了三种不同的方法，用于在Java中查找与原始数字具有相同位数的下一个更大数字。总的来说，双指针方法在效率和简便性之间取得了良好的平衡，可用于查找具有相同位数的下一个更大数字。