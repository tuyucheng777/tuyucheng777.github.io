---
layout: post
title:  如何在Java中查找第K个最大元素
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 简介

在本文中，我们将介绍在唯一数字序列中查找第k大元素的各种解决方案，我们将使用一个整数数组作为示例。

我们还将讨论每种算法的平均和最坏情况下的时间复杂度。

## 2. 解决方案

现在让我们探讨一些可能的解决方案-一种使用普通排序，另外两种使用从快速排序派生的快速选择算法。

### 2.1 排序

当我们思考这个问题时，**可能想到的最明显的解决方案就是对数组进行排序**。

让我们定义所需的步骤：

-   对数组进行升序排序
-   由于数组的最后一个元素将是最大的元素，因此第k个最大的元素将位于第x个索引处，其中x = length(array) – k

可以看出，解决方案很简单，但需要对整个数组进行排序。因此，时间复杂度将为O(n * logn)：

```java
public int findKthLargestBySorting(Integer[] arr, int k) {
    Arrays.sort(arr);
    int targetIndex = arr.length - k;
    return arr[targetIndex];
}
```

另一种方法是按降序对数组进行排序，并简单地返回第(k-1)个索引上的元素：

```java
public int findKthLargestBySortingDesc(Integer[] arr, int k) {
    Arrays.sort(arr, Collections.reverseOrder());
    return arr[k-1];
}
```

### 2.2 快速选择

这可以被认为是对之前方法的优化，在这里，我们选择[快速排序](https://www.geeksforgeeks.org/quick-sort/)进行排序。分析问题陈述，**我们意识到实际上并不需要对整个数组进行排序-我们只需要重新排列其内容，使数组的第k个元素成为第k个最大或最小的元素**。

在快速排序中，我们选择一个枢轴元素并将其移动到正确的位置，我们还围绕它对数组进行分区。**在快速选择中，我们的想法是，当枢轴元素本身是第k个最大元素时，停止排序**。

如果我们不对枢轴的左右两侧都进行递归，我们可以进一步优化算法，我们只需要根据枢轴的位置对其中一侧进行递归。

我们先看一下快速选择算法的基本思想：

-   选择一个枢轴元素并相应地对数组进行分区
    -   选择最右边的元素作为枢轴
    -   重新排列数组，使枢轴元素放置在正确的位置-所有小于枢轴的元素将位于较低的索引处，而大于枢轴的元素将位于高于枢轴的索引处
-   如果枢轴位于数组中的第k个元素，则退出该过程，因为枢轴是第k个最大的元素
-   如果枢轴位置大于k，则继续使用左子数组进行该过程，否则，重复使用右子数组进行该过程

我们可以编写通用逻辑，它也可用于查找第k个最小元素。我们将定义一个方法findKthElementByQuickSelect()，它将返回排序数组中的第k个元素。

如果我们按升序对数组进行排序，则数组的第k个元素将是第k个最小的元素。要找到第k个最大的元素，我们可以传递k = length(Array) – k。

让我们实现这个解决方案：

```java
public int findKthElementByQuickSelect(Integer[] arr, int left, int right, int k) {
    if (k >= 0 && k <= right - left + 1) {
        int pos = partition(arr, left, right);
        if (pos - left == k) {
            return arr[pos];
        }
        if (pos - left > k) {
            return findKthElementByQuickSelect(arr, left, pos - 1, k);
        }
        return findKthElementByQuickSelect(arr, pos + 1,
                right, k - pos + left - 1);
    }
    return 0;
}
```

现在让我们实现partition方法，该方法选择最右边的元素作为枢轴，将其放在适当的索引处，并以较低索引处的元素应小于枢轴元素的方式对数组进行分区。

类似地，索引较高的元素将大于枢轴元素：

```java
public int partition(Integer[] arr, int left, int right) {
    int pivot = arr[right];
    Integer[] leftArr;
    Integer[] rightArr;

    leftArr = IntStream.range(left, right)
            .filter(i -> arr[i] < pivot)
            .map(i -> arr[i])
            .boxed()
            .toArray(Integer[]::new);

    rightArr = IntStream.range(left, right)
            .filter(i -> arr[i] > pivot)
            .map(i -> arr[i])
            .boxed()
            .toArray(Integer[]::new);

    int leftArraySize = leftArr.length;
    System.arraycopy(leftArr, 0, arr, left, leftArraySize);
    arr[leftArraySize+left] = pivot;
    System.arraycopy(rightArr, 0, arr, left + leftArraySize + 1, rightArr.length);

    return left + leftArraySize;
}
```

有一种更简单的迭代方法来实现分区：

```java
public int partitionIterative(Integer[] arr, int left, int right) {
    int pivot = arr[right], i = left;
    for (int j = left; j <= right - 1; j++) {
        if (arr[j] <= pivot) {
            swap(arr, i, j);
            i++;
        }
    }
    swap(arr, i, right);
    return i;
}

public void swap(Integer[] arr, int n1, int n2) {
    int temp = arr[n2];
    arr[n2] = arr[n1];
    arr[n1] = temp;
}
```

该解决方案平均时间为O(n)，但是，在最坏的情况下，时间复杂度将达到O(n^2)。

### 2.3 随机分区的快速选择

这种方法是对之前方法的轻微修改，如果数组几乎/完全排序并且我们选择最右边的元素作为枢轴，则左右子数组的划分将非常不均匀。

此方法建议**以随机方式选择初始枢轴元素**，不过，我们不需要更改分区逻辑。

我们不调用partition，而是调用randomPartition方法，该方法选择一个随机元素并将其与最右边的元素交换，最后调用partition方法。

让我们实现randomPartition方法：

```java
public int randomPartition(Integer arr[], int left, int right) {
    int n = right - left + 1;
    int pivot = (int) (Math.random()) * n;
    swap(arr, left + pivot, right);
    return partition(arr, left, right);
}
```

在大多数情况下，此解决方案比前一种情况效果更好。

随机快速选择的预期时间复杂度为O(n)。

然而，最坏的时间复杂度仍然是O(n^2)。

## 3. 不排序，找到第二大元素

在查找数组中的第2大元素时，我们不需要对整个数组进行排序，我们只需要遍历数组中的所有元素并保留第2大的元素即可。即便如此，当我们想要查找第2大元素时，我们也需要在值的子集中保留或识别最大的元素。

我们可以通过创建一个大小为2的结果数组来查找数组中的第2大元素，该数组仅保留最大元素和第2大元素，结果数组将第2大元素保存在索引位置1处。因此，我们迭代输入数组，并将其每个元素与结果数组中的最大元素和第2大元素进行比较，当输入数组元素为最大元素或第2大元素时，我们会在迭代过程中更新结果数组。最终，我们从结果数组中返回第2大元素。

为了演示，让我们使用一个接收Integer数组并返回int的方法：

```java
public int findSecondLargestWithoutSorting(Integer[] arr) throws Exception {
    Integer[] result = new Integer[2];

    if (arr == null || arr.length < 2) { 
        throw new Exception("Array should have at least two elements and be not null"); 
    } else { 
          if (arr[0] > arr[1]) {
              result[0] = arr[0];
              result[1] = arr[1];
          } else {
                result[0] = arr[1];
                result[1] = arr[0];
            }
          if (arr.length > 2) {
              for (int i = 2; i < arr.length; i++) { 
                  if (arr[i] > result[0]) {
                      result[1] = result[0];
                      result[0] = arr[i];

                  } else if (arr[i] > result[1]) {
                        result[1] = arr[i];
                    }
              }
          }
      }   

    return result[1];
}
```

我们使用JUnit 5集成测试来验证该方法是否返回整数数组中的第2大元素：

```java
@Test
void givenIntArray_whenFindSecondLargestWithoutSorting_thenGetResult() throws Exception{
    assertThat(findKthLargest.findSecondLargestWithoutSorting(arr)).isEqualTo(9);
}
```

JUnit测试应该通过示例输入数组{3, 7, 1, 2, 8, 10, 4, 5, 6, 9}。

## 4. 总结

在本文中，我们讨论了在一个包含唯一数字的数组中查找第k个最大(或最小)元素的不同解决方案；最简单的解决方案是对数组进行排序并返回第k个元素，该解决方案的时间复杂度为O(n*logn)。

我们还讨论了快速选择的两种变体，该算法并不简单，但在一般情况下具有O(n)的时间复杂度。