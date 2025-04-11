---
layout: post
title:  在Java中查找两个排序数组中的第K个最小元素
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 简介

在本文中，我们将了解如何**在两个已排序数组的并集中找到第k小的元素**。

首先，我们将定义确切的问题。其次，我们将讨论两个低效但简单的解决方案。第三，我们将讨论一个基于对两个数组进行二分查找的高效解决方案。最后，我们将进行一些测试来验证我们的算法是否有效。

我们还将看到算法所有部分的Java代码片段，**为简单起见，我们的实现将仅对整数进行操作**。但是，所描述的算法适用于所有可比较的数据类型，甚至可以使用泛型来实现。

## 2. 两个排序数组的并集的第K小元素是什么？

### 2.1 第K个最小元素

为了找到数组中第k个最小元素(也称为k阶统计量)，我们通常使用[选择算法](https://www.baeldung.com/java-kth-largest-element)。**但是，这些算法对单个未排序的数组进行操作，而在本文中，我们想要在两个排序数组中找到第k个最小元素**。

在讨论这个问题的几种解决方案之前，我们先明确一下我们想要实现的目标。为此，我们先来看一个例子。

我们给出两个排序数组(a和b)，它们不一定需要具有相同数量的元素：

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays01.png)

在这两个数组中，我们想要找到第k个最小元素。更具体地说，我们想在合并并排序后的数组中找到第k个最小的元素：

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays02.png)

我们示例中合并并排序后的数组如(c)所示，第1个最小元素是3，第4个最小元素是20。

### 2.2 重复值

我们还需要定义如何处理重复值，一个元素可能在其中一个数组中出现多次(例如数组a中的元素3)，也可能在第二个数组(b)中再次出现。

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays03.png)

如果我们只计算一次重复项，则计算方式如(c)所示；如果我们计算某个元素的所有出现次数，则计算方式如(d)所示。

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays04.png)

在本文的剩余部分，我们将像(d)中所示的那样计算重复项，从而将它们视为不同的元素进行计算。

## 3. 两种简单但效率较低的方法

### 3.1 拼接两个数组并进行排序

找到第k个最小元素的最简单方法是拼接数组，对它们进行排序，然后返回结果数组的第k个元素：
```java
int getKthElementSorted(int[] list1, int[] list2, int k) {
    int length1 = list1.length, length2 = list2.length;
    int[] combinedArray = new int[length1 + length2];
    System.arraycopy(list1, 0, combinedArray, 0, list1.length);
    System.arraycopy(list2, 0, combinedArray, list1.length, list2.length);
    Arrays.sort(combinedArray);

    return combinedArray[k-1];
}
```

其中n为第一个数组的长度，m为第二个数组的长度，我们得到组合长度c= n + m。

由于排序的复杂度为O(c log c)，因此该方法的总体复杂度为O(n log n)。

这种方法的缺点是我们需要创建数组的副本，这会导致需要更多的空间。

### 3.2 合并两个数组

与[归并排序](https://www.baeldung.com/java-merge-sort)算法的一个步骤类似，我们可以[合并](https://www.baeldung.com/java-merge-sorted-arrays)两个数组，然后直接检索第k个元素。

合并算法的基本思想是从两个指针开始，分别指向第一个和第二个数组的第一个元素(a)。

然后，我们比较指针处的两个元素(3和4)，将较小的元素(3)添加到结果中，并将指针向前移动一个位置(b)。再次，我们比较指针处的元素，并将较小的元素(4)添加到结果中。

我们以相同的方式继续操作，直到所有元素都添加到结果数组中。如果其中一个输入数组没有更多元素，我们只需将另一个输入数组的所有剩余元素复制到结果数组中。

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays05.png)

**如果我们不复制整个数组，而是在结果数组包含k个元素时停止，则可以提高性能。我们甚至不需要为合并后的数组创建额外的数组，只需对原始数组进行操作即可**。

以下是Java中的实现：
```java
public static int getKthElementMerge(int[] list1, int[] list2, int k) {
    int i1 = 0, i2 = 0;

    while(i1 < list1.length && i2 < list2.length && (i1 + i2) < k) {
        if(list1[i1] < list2[i2]) {
            i1++;
        } else {
            i2++;
        }
    }

    if((i1 + i2) < k) {
        return i1 < list1.length ? list1[k - i2 - 1] : list2[k - i1 - 1];
    } else if(i1 > 0 && i2 > 0) {
        return Math.max(list1[i1-1], list2[i2-1]);
    } else {
        return i1 == 0 ? list2[i2-1] : list1[i1-1];
    }
}
```

很容易理解，该算法的时间复杂度为O(k)，**该算法的一个优点是，它可以很容易地调整为只考虑重复元素一次**。

## 4. 对两个数组进行二分查找

我们能做得比O(k)更好吗？答案是可以的，基本思想是对两个数组进行二分搜索算法。

为了实现这一点，我们需要一个能够以恒定时间读取其所有元素的数据结构。在Java中，这个数据结构可以是数组或ArrayList。

让我们定义要实现的方法的骨架：
```java
int findKthElement(int k, int[] list1, int[] list2) throws NoSuchElementException, IllegalArgumentException {

    // check input (see below)

    // handle special cases (see below)

    // binary search (see below)
}
```

在这里，我们将k和两个数组作为参数传递。首先，我们将验证输入；其次，我们将处理一些特殊情况，然后进行二分查找。在接下来的三节中，我们将以相反的顺序介绍这三个步骤，首先，我们将讨论二分查找，其次，我们将讨论特殊情况，最后，我们将讨论参数验证。

### 4.1 二分查找

标准二分查找(即查找特定元素)有两种可能的结果：要么找到了目标元素，查找成功；要么没找到，查找失败。**而我们这里的情况有所不同，我们想要查找第k个最小元素。在这种情况下，我们总会得到一个结果**。

让我们看看如何实现这一点。

#### 4.1.1 从两个数组中找出正确的元素数量

我们从第一个数组中的一定数量的元素开始搜索，我们把这个数字称为nElementsList1。由于我们总共需要k个元素，因此nElementsList1的数量为：
```java
int nElementsList2 = k - nElementsList1;
```

举个例子，假设k = 8，我们从第一个数组中的4个元素和第二个数组中的4个元素开始(a)。

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays06.png)

如果第一个数组中的第4个元素大于第二个数组中的第4个元素，则说明我们从第一个数组中取出了太多元素，可以减少nElementsList1(b)。否则，说明我们从第一个数组中取出了太少元素，可以增加nElementsList1(b')。

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays07.png)

我们继续，直到达到停止标准。在讨论停止标准之前，我们先来看看到目前为止所描述的代码：
```java
int right = k;
int left = = 0;
do {
    nElementsList1 = ((left + right) / 2) + 1;
    nElementsList2 = k - nElementsList1;

    if(nElementsList2 > 0) {
        if (list1[nElementsList1 - 1] > list2[nElementsList2 - 1]) {
            right = nElementsList1 - 2;
        } else {
            left = nElementsList1;
        }
    }
} while(!kthSmallesElementFound(list1, list2, nElementsList1, nElementsList2));
```

#### 4.1.2 停止标准

我们可以在两种情况下停止。第一种情况，如果我们从第一个数组中取出的最大元素等于从第二个数组中取出的最大元素(c)，则停止。在这种情况下，我们可以直接返回该元素。

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays08.png)

其次，如果满足以下两个条件，我们就可以停止(d)：

- 从第一个数组中取出的最大元素小于我们不从第二个数组中取出的最小元素(11 < 100)
- 从第二个数组中取出的最大元素小于我们不从第一个数组中取出的最小元素(21 < 27)

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays09.png)

很容易想象(d')为什么该条件有效：我们从两个数组中取出的所有元素肯定比两个数组中的任何其他元素都小。

以下是停止标准的代码：
```java
private static boolean foundCorrectNumberOfElementsInBothLists(int[] list1, int[] list2, int nElementsList1, int nElementsList2) {
    // we do not take any element from the second list
    if(nElementsList2 < 1) {
        return true;
    }

    if(list1[nElementsList1-1] == list2[nElementsList2-1]) {
        return true;
    }

    if(nElementsList1 == list1.length) {
        return list1[nElementsList1-1] <= list2[nElementsList2];
    }

    if(nElementsList2 == list2.length) {
        return list2[nElementsList2-1] <= list1[nElementsList1];
    }

    return list1[nElementsList1-1] <= list2[nElementsList2] && list2[nElementsList2-1] <= list1[nElementsList1];
}
```

#### 4.1.3 返回值

最后，我们需要返回正确的值，这里有三种可能的情况：

- 我们不从第二个数组中获取任何元素，因此目标值在第一个数组中(e)
- 目标值位于第一个数组(e')中
- 目标值位于第二个数组(e”)中

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays10.png)

让我们通过代码看一下：
```java
return nElementsList2 == 0 ? list1[nElementsList1-1] : max(list1[nElementsList1-1], list2[nElementsList2-1]);
```

请注意，我们不需要处理不从第一个数组中获取任何元素的情况-我们将在稍后处理特殊情况时排除这种情况。

### 4.2 左右边界的初始值

到目前为止，我们用k和0初始化了第一个数组的右边界和左边界：
```java
int right = k;
int left = 0;
```

但是，根据k的值，我们需要调整这些边界。

首先，如果k超出了第一个数组的长度，我们需要将最后一个元素作为右边界。这样做的原因很简单，因为我们不能从数组中取出超过数组元素数量的元素。

其次，如果k大于第二个数组中的元素数量，我们就可以确定至少需要从第一个数组中取出(k– length(list2))个元素。例如，假设k = 7，由于第二个数组只有4个元素，我们知道至少需要从第一个数组中取出3个元素，因此我们可以将L设置为2：

![](/assets/images/2025/algorithms/javakthsmallestelementinsortedarrays11.png)

以下是适配的左右边界的代码：
```java
// correct left boundary if k is bigger than the size of list2
int left = k < list2.length ? 0 : k - list2.length - 1;

// the inital right boundary cannot exceed the list1
int right = min(k-1, list1.length - 1);
```

### 4.3 特殊情况处理

在进行实际的二分查找之前，我们可以处理一些特殊情况，以使算法稍微简单一些，并避免异常。以下是代码，注释中有相关解释：
```java
// we are looking for the minimum value
if(k == 1) {
    return min(list1[0], list2[0]);
}

// we are looking for the maximum value
if(list1.length + list2.length == k) {
    return max(list1[list1.length-1], list2[list2.length-1]);
}

// swap lists if needed to make sure we take at least one element from list1
if(k <= list2.length && list2[k-1] < list1[0]) {
    int[] list1_ = list1;
    list1 = list2;
    list2 = list1_;
}
```

### 4.4 输入验证

我们先来看看输入验证，为了防止算法失败并抛出NullPointerException或ArrayIndexOutOfBoundsException等异常，我们要确保3个参数满足以下条件：

- 两个数组均不能为空，并且至少有一个元素
- k必须>=0，并且不能大于两个数组的长度之和

以下是我们的代码验证：
```java
void checkInput(int k, int[] list1, int[] list2) throws NoSuchElementException, IllegalArgumentException {
    if(list1 == null || list2 == null || k < 1) {
        throw new IllegalArgumentException();
    }

    if(list1.length == 0 || list2.length == 0) {
        throw new IllegalArgumentException();
    }

    if(k > list1.length + list2.length) {
        throw new NoSuchElementException();
    }
}
```

### 4.5 完整代码

以下是我们刚刚描述的算法的完整代码：
```java
public static int findKthElement(int k, int[] list1, int[] list2) throws NoSuchElementException, IllegalArgumentException {
    checkInput(k, list1, list2);

    // we are looking for the minimum value
    if(k == 1) {
        return min(list1[0], list2[0]);
    }

    // we are looking for the maximum value
    if(list1.length + list2.length == k) {
        return max(list1[list1.length-1], list2[list2.length-1]);
    }

    // swap lists if needed to make sure we take at least one element from list1
    if(k <= list2.length && list2[k-1] < list1[0]) {
        int[] list1_ = list1;
        list1 = list2;
        list2 = list1_;
    }

    // correct left boundary if k is bigger than the size of list2
    int left = k < list2.length ? 0 : k - list2.length - 1;

    // the inital right boundary cannot exceed the list1 
    int right = min(k-1, list1.length - 1);

    int nElementsList1, nElementsList2;

    // binary search 
    do {
        nElementsList1 = ((left + right) / 2) + 1;
        nElementsList2 = k - nElementsList1;

        if(nElementsList2 > 0) {
            if (list1[nElementsList1 - 1] > list2[nElementsList2 - 1]) {
                right = nElementsList1 - 2;
            } else {
                left = nElementsList1;
            }
        }
    } while(!kthSmallesElementFound(list1, list2, nElementsList1, nElementsList2));

    return nElementsList2 == 0 ? list1[nElementsList1-1] : max(list1[nElementsList1-1], list2[nElementsList2-1]);
}

private static boolean foundCorrectNumberOfElementsInBothLists(int[] list1, int[] list2, int nElementsList1, int nElementsList2) {
    // we do not take any element from the second list
    if(nElementsList2 < 1) {
        return true;
    }

    if(list1[nElementsList1-1] == list2[nElementsList2-1]) {
        return true;
    }

    if(nElementsList1 == list1.length) {
        return list1[nElementsList1-1] <= list2[nElementsList2];
    }

    if(nElementsList2 == list2.length) {
        return list2[nElementsList2-1] <= list1[nElementsList1];
    }

    return list1[nElementsList1-1] <= list2[nElementsList2] && list2[nElementsList2-1] <= list1[nElementsList1];
}
```

## 5. 测试算法

在我们的GitHub仓库中，有许多测试用例，涵盖了许多可能的输入数组以及许多极端情况。

这里我们只指出其中一项测试，它不是针对静态输入数组进行测试，而是将双二分查找算法的结果与简单的拼接排序算法的结果进行比较。输入由两个随机数组组成：
```java
int[] sortedRandomIntArrayOfLength(int length) {
    int[] intArray = new Random().ints(length).toArray();
    Arrays.sort(intArray);
    return intArray;
}
```

以下方法执行一次单一测试：
```java
private void random() {
    Random random = new Random();
    int length1 = (Math.abs(random.nextInt())) % 1000 + 1;
    int length2 = (Math.abs(random.nextInt())) % 1000 + 1;

    int[] list1 = sortedRandomIntArrayOfLength(length1);
    int[] list2 = sortedRandomIntArrayOfLength(length2);

    int k = (Math.abs(random.nextInt()) + 1) % (length1 + length2);

    int result = findKthElement(k, list1, list2);
    int result2 = getKthElementSorted(list1, list2, k);
    int result3 = getKthElementMerge(list1, list2, k);

    assertEquals(result2, result);
    assertEquals(result2, result3);
}
```

我们可以调用上述方法来运行大量的测试，如下所示：
```java
@Test
void randomTests() {
    IntStream.range(1, 100000).forEach(i -> random());
}
```

## 6. 总结

在本文中，我们了解了在两个已排序数组的并集中查找第k个最小元素的几种方法。首先，我们了解了一种简单直接的O(n log n)算法，然后是复杂度为O(n)的版本，最后是一个运行时间为(log n)的算法。

我们看到的最后一个算法是一个很好的理论练习；然而，在大多数实际应用中，我们应该考虑使用前两种算法中的一种，它们比对两个数组进行二分查找要简单得多。当然，如果性能是一个问题，二分查找可能是一个解决方案。