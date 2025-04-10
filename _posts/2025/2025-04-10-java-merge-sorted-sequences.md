---
layout: post
title:  高效合并已排序的Java序列
category: algorithms
copyright: algorithms
excerpt: 合并
---

## 1. 概述

在这个简短的教程中，我们将看到如何使用堆有效地合并已排序的数组。

## 2. 算法

由于我们的问题陈述是使用堆来合并数组，因此我们将使用最小堆来解决我们的问题。**最小堆只不过是一棵二叉树，其中每个节点的值都小于其子节点的值**。

通常，最小堆是使用数组来实现的，其中数组在查找节点的父节点和子节点时满足特定规则。

对于数组A[]和索引i处的元素：

- A[(i-1)/2\]将返回其父级
- A[(2\*i)+1\]将返回左子节点
- A[(2\*i)+2\]将返回右子节点

这是最小堆及其数组表示的图片：

![](/assets/images/2025/algorithms/javamergesortedsequences01.png)

现在让我们创建合并一组排序数组的算法：

1. 创建一个数组来存储结果，其大小由所有输入数组的长度之和决定。
2. 创建大小等于输入数组数量的第二个数组，并用所有输入数组的第一个元素填充它。
3. 通过对所有节点及其子节点应用最小堆规则，将先前创建的数组转换为最小堆。
4. 重复以下步骤，直到结果数组完全填充。
5. 从最小堆中获取根元素并将其存储在结果数组中。
6. 将根元素替换为当前根所在的数组中的下一个元素。
7. 在我们的最小堆数组上再次应用最小堆规则。

我们的算法**有一个递归流程来创建最小堆**，并且我们必须访问输入数组的所有元素。

该算法的[时间复杂度](https://www.baeldung.com/java-algorithm-complexity)为O(k log n)，其中k是所有输入数组中元素的总数，n是已排序数组的总数。

现在让我们看一个示例输入和运行算法后的预期结果，以便我们更好地理解问题，对于这些数组：
```text
{ { 0, 6 }, { 1, 5, 10, 100 }, { 2, 4, 200, 650 } }
```

该算法应该返回一个结果数组：
```text
{ 0, 1, 2, 4, 5, 6, 10, 100, 200, 650 }
```

## 3. Java实现

现在我们已经对最小堆是什么以及合并算法的工作原理有了基本的了解，让我们看看Java实现，我们将使用两个类-一个用于表示堆节点，另一个用于实现合并算法。

### 3.1 堆节点表示

在实现算法本身之前，让我们创建一个表示堆节点的类，它将存储节点值和两个支持字段：
```java
public class HeapNode {

    int element;
    int arrayIndex;
    int nextElementIndex = 1;

    public HeapNode(int element, int arrayIndex) {
        this.element = element;
        this.arrayIndex = arrayIndex;
    }
}
```

请注意，为了简单起见，我们特意省略了Getter和Setter。我们将使用arrayIndex属性来存储当前堆节点元素所在的数组的索引，我们将使用nextElementIndex属性来存储在将根节点移动到结果数组后要获取的元素的索引。

最初，nextElementIndex的值为1，在替换最小堆的根节点后，我们将增加其值。

### 3.2 最小堆合并算法

我们的下一个类是表示最小堆本身并实现合并算法：
```java
public class MinHeap {

    HeapNode[] heapNodes;

    public MinHeap(HeapNode heapNodes[]) {
        this.heapNodes = heapNodes;
        heapifyFromLastLeafsParent();
    }

    int getParentNodeIndex(int index) {
        return (index - 1) / 2;
    }

    int getLeftNodeIndex(int index) {
        return (2 * index + 1);
    }

    int getRightNodeIndex(int index) {
        return (2 * index + 2);
    }

    HeapNode getRootNode() {
        return heapNodes[0];
    }

    // additional implementation methods
}
```

现在我们已经创建了最小堆类，让我们添加一个方法来堆化子树，其中子树的根节点位于数组的给定索引处：
```java
void heapify(int index) {
    int leftNodeIndex = getLeftNodeIndex(index);
    int rightNodeIndex = getRightNodeIndex(index);
    int smallestElementIndex = index;
    if (leftNodeIndex < heapNodes.length
            && heapNodes[leftNodeIndex].element < heapNodes[index].element) {
        smallestElementIndex = leftNodeIndex;
    }
    if (rightNodeIndex < heapNodes.length
            && heapNodes[rightNodeIndex].element < heapNodes[smallestElementIndex].element) {
        smallestElementIndex = rightNodeIndex;
    }
    if (smallestElementIndex != index) {
        swap(index, smallestElementIndex);
        heapify(smallestElementIndex);
    }
}
```

当我们使用数组表示最小堆时，最后一个叶子节点始终位于数组的末尾。因此，当通过迭代调用heapify()方法将数组转换为最小堆时，我们只需要从最后一个叶子的父节点开始迭代：
```java
void heapifyFromLastLeafsParent() {
    int lastLeafsParentIndex = getParentNodeIndex(heapNodes.length);
    while (lastLeafsParentIndex >= 0) {
        heapify(lastLeafsParentIndex);
        lastLeafsParentIndex--;
    }
}
```

我们的下一个方法将实际实现我们的算法，为了更好地理解，我们将该方法分为两部分，看看它是如何工作的：
```java
int[] merge(int[][] array) {
    // transform input arrays
    // run the minheap algorithm
    // return the resulting array
}
```

第一部分将输入数组转换为包含第一个数组所有元素的堆节点数组，并找到结果数组的大小：
```java
HeapNode[] heapNodes = new HeapNode[array.length];
int resultingArraySize = 0;

for (int i = 0; i < array.length; i++) {
    HeapNode node = new HeapNode(array[i][0], i);
    heapNodes[i] = node;
    resultingArraySize += array[i].length;
}
```

下一部分通过实现算法的第4、5、6和7步来填充结果数组：
```java
MinHeap minHeap = new MinHeap(heapNodes);
int[] resultingArray = new int[resultingArraySize];

for (int i = 0; i < resultingArraySize; i++) {
    HeapNode root = minHeap.getRootNode();
    resultingArray[i] = root.element;

    if (root.nextElementIndex < array[root.arrayIndex].length) {
        root.element = array[root.arrayIndex][root.nextElementIndex++];
    } else {
        root.element = Integer.MAX_VALUE;
    }
    minHeap.heapify(0);
}
```

## 4. 测试算法

现在让我们用前面提到的相同输入来测试我们的算法：
```java
int[][] inputArray = { { 0, 6 }, { 1, 5, 10, 100 }, { 2, 4, 200, 650 } };
int[] expectedArray = { 0, 1, 2, 4, 5, 6, 10, 100, 200, 650 };

int[] resultArray = MinHeap.merge(inputArray);

assertThat(resultArray.length, is(equalTo(10)));
assertThat(resultArray, is(equalTo(expectedArray)));
```

## 5. 总结

在本教程中，我们学习了如何使用最小堆有效地合并已排序数组。