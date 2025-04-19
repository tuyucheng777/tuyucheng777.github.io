---
layout: post
title:  查找列表的峰值元素
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 简介

数组中的峰值元素对于许多算法都至关重要，它们能够帮助我们深入了解数据集的特征。在本教程中，我们将探讨峰值元素的概念，解释其重要性，并探索在单峰和多峰场景下识别峰值元素的有效方法。

## 2. 什么是峰值元素？

**数组中的峰值元素定义为严格大于其相邻元素的元素**，如果边缘元素大于其唯一相邻元素，则认为它们处于峰值位置。

在元素相等的情况下，严格的峰值并不存在。相反，峰值是指元素首次超过其相邻元素的情况。

### 2.1 示例

为了更好地理解峰值元素的概念，请看以下示例：

例1：

```text
List: [1, 2, 20, 3, 1, 0]
Peak Element: 20
```

这里，20是一个峰值，因为它大于其相邻元素。

示例2：

```text
List: [5, 13, 15, 25, 40, 75, 100]
Peak Element: 100
```

100是一个峰值，因为它大于75，并且右侧没有元素。

示例3：

```text
List: [9, 30, 13, 2, 23, 104, 67, 12]
Peak Element: 30 or 104, as both are valid peaks
```

30和104都可视为峰值。

## 3. 查找单峰元素

当数组仅包含一个峰值元素时，一种直接的方法是使用[线性搜索](https://www.baeldung.com/cs/linear-search-faster)。**该算法扫描数组元素，将每个元素与其相邻元素进行比较，直到找到峰值**。该方法的时间复杂度为O(n)，其中n是数组的大小。

```java
public class SinglePeakFinder {
    public static OptionalInt findSinglePeak(int[] arr) {
        int n = arr.length;

        if (n < 2) {
            return n == 0 ? OptionalInt.empty() : OptionalInt.of(arr[0]);
        }

        if (arr[0] >= arr[1]) {
            return OptionalInt.of(arr[0]);
        }

        for (int i = 1; i < n - 1; i++) {
            if (arr[i] >= arr[i - 1] && arr[i] >= arr[i + 1]) {
                return OptionalInt.of(arr[i]);
            }
        }

        if (arr[n - 1] >= arr[n - 2]) {
            return OptionalInt.of(arr[n - 1]);
        }

        return OptionalInt.empty();
    }
}
```

该算法从索引1到n-2遍历数组，检查当前元素是否大于其相邻元素。如果找到峰值，则返回一个包含该峰值的[OptionalInt](https://www.baeldung.com/java-optional)类型。此外，该算法还处理峰值位于数组两端的极端情况。

```java
public class SinglePeakFinderUnitTest {
    @Test
    void findSinglePeak_givenArrayOfIntegers_whenValidInput_thenReturnsCorrectPeak() {
        int[] arr = {0, 10, 2, 4, 5, 1};
        OptionalInt peak = SinglePeakFinder.findSinglePeak(arr);
        assertTrue(peak.isPresent());
        assertEquals(10, peak.getAsInt());
    }

    @Test
    void findSinglePeak_givenEmptyArray_thenReturnsEmptyOptional() {
        int[] arr = {};
        OptionalInt peak = SinglePeakFinder.findSinglePeak(arr);
        assertTrue(peak.isEmpty());
    }

    @Test
    void findSinglePeak_givenEqualElementArray_thenReturnsCorrectPeak() {
        int[] arr = {-2, -2, -2, -2, -2};
        OptionalInt peak = SinglePeakFinder.findSinglePeak(arr);
        assertTrue(peak.isPresent());
        assertEquals(-2, peak.getAsInt());
    }
}
```

对于双调数组(其特征是先单调递增序列，后跟单调递减序列)，可以更高效地找到峰值。**通过应用改进的二分查找技术，我们可以在O(log n)时间内找到峰值，从而显著降低复杂度**。

需要注意的是，确定数组是否为双调数组需要进行检查，在最坏的情况下，这可能需要接近线性时间。因此，当数组的双调性质已知时，[二分查找方法](https://www.baeldung.com/java-binary-search)的效率提升最为显著。

## 4. 寻找多个峰值元素

识别数组中的多个峰值元素通常需要检查每个元素与其相邻元素的关系，从而需要一个时间复杂度为O(n)的线性搜索算法。这种方法确保不会忽略任何潜在的峰值，因此适用于一般数组。

在特定场景下，当数组结构允许分割成可预测的模式时，可以应用改进的二分查找技术来更有效地查找峰值；让我们使用改进的二分查找算法来实现O(log n)的时间复杂度。

**算法解释**：

- **初始化指针**：从两个指针low和high开始，表示数组的范围。

- **二分查找**：计算当前范围的中间索引mid。

- **将中间值与邻居进行比较**：检查索引中间的元素是否大于其邻居。

  - 如果为true，则mid为峰值。
  - 如果为false，则向具有更大邻居的一侧移动，确保我们向潜在的峰值移动。

- **重复**：继续该过程，直到范围缩小到单个元素。

```java
public class MultiplePeakFinder {
    public static List<Integer> findPeaks(int[] arr) {
        List<Integer> peaks = new ArrayList<>();

        if (arr == null || arr.length == 0) {
            return peaks;
        }
        findPeakElements(arr, 0, arr.length - 1, peaks, arr.length);
        return peaks;
    }

    private static void findPeakElements(int[] arr, int low, int high, List<Integer> peaks, int length) {
        if (low > high) {
            return;
        }

        int mid = low + (high - low) / 2;

        boolean isPeak = (mid == 0 || arr[mid] > arr[mid - 1]) && (mid == length - 1 || arr[mid] > arr[mid + 1]);
        boolean isFirstInSequence = mid > 0 && arr[mid] == arr[mid - 1] && arr[mid] > arr[mid + 1];

        if (isPeak || isFirstInSequence) {

            if (!peaks.contains(arr[mid])) {
                peaks.add(arr[mid]);
            }
        }

        findPeakElements(arr, low, mid - 1, peaks, length);
        findPeakElements(arr, mid + 1, high, peaks, length);
    }
}
```

MultiplePeakFinder类采用改进的二分查找算法，可以高效地识别数组中的多个峰值元素。**findPeaks方法初始化两个指针low和high，表示数组的范围**。

它计算中间索引(mid)，并检查位于mid的元素是否大于其相邻元素。如果为true，则将mid标记为峰值，并继续在潜在的峰值丰富区域中进行搜索。

```java
public class MultiplePeakFinderUnitTest {
    @Test
    void findPeaks_givenArrayOfIntegers_whenValidInput_thenReturnsCorrectPeaks() {

        MultiplePeakFinder finder = new MultiplePeakFinder();
        int[] array = {1, 13, 7, 0, 4, 1, 4, 45, 50};
        List<Integer> peaks = finder.findPeaks(array);

        assertEquals(3, peaks.size());
        assertTrue(peaks.contains(4));
        assertTrue(peaks.contains(13));
        assertTrue(peaks.contains(50));
    }
}
```

**二分查找查找峰值的效率取决于数组的结构，因此无需检查每个元素即可检测峰值**。然而，如果不知道数组的结构，或者缺乏合适的二分查找模式，线性查找是最可靠的方法，可以保证不会遗漏任何峰值。

## 5. 处理边缘情况

理解和解决边缘情况对于确保峰值元算法的稳健性和可靠性至关重要。

### 5.1 无峰数组

**在数组不包含峰值元素的情况下，必须指出这种缺失**。当未找到峰值时，返回一个空数组：

```java
public class PeakElementFinder {
    public List<Integer> findPeakElements(int[] arr) {
        int n = arr.length;
        List<Integer> peaks = new ArrayList<>();

        if (n == 0) {
            return peaks;
        }

        for (int i = 0; i < n; i++) {
            if (isPeak(arr, i, n)) {
                peaks.add(arr[i]);
            }
        }

        return peaks;
    }

    private boolean isPeak(int[] arr, int index, int n) {
        return arr[index] >= arr[index - 1] && arr[index] >= arr[index + 1];
    }
}
```

findPeakElement方法遍历数组，并利用isPeak辅助函数来识别峰值。如果没有找到峰值，则返回一个空数组。

```java
public class PeakElementFinderUnitTest {
    @Test
    void findPeakElement_givenArrayOfIntegers_whenValidInput_thenReturnsCorrectPeak() {
        PeakElementFinder finder = new PeakElementFinder();
        int[] array = {1, 2, 3, 2, 1};
        List<Integer> peaks = finder.findPeakElements(array);
        assertEquals(1, peaks.size());
        assertTrue(peaks.contains(3));
    }

    @Test
    void findPeakElement_givenArrayOfIntegers_whenNoPeaks_thenReturnsEmptyList() {
        PeakElementFinder finder = new PeakElementFinder();
        int[] array = {};
        List<Integer> peaks = finder.findPeakElements(array);
        assertEquals(0, peaks.size());
    }
}
```

### 5.2 具有极值峰值的数组

**当峰值出现在第一个或最后一个元素时，需要进行特殊处理，以避免未定义的邻居比较**。让我们在isPeak方法中添加一个[条件检查](https://www.baeldung.com/java-multiple-or-conditions-if-statement)来处理这些情况：

```java
private boolean isPeak(int[] arr, int index, int n) {
    if (index == 0) {
        return n > 1 ? arr[index] >= arr[index + 1] : true;
    } else if (index == n - 1) {
        return arr[index] >= arr[index - 1];
    }

    return arr[index] >= arr[index - 1] && arr[index] >= arr[index + 1];
}
```

这种修改确保正确识别极值的峰值，而无需尝试与未定义的邻居进行比较。

```java
public class PeakElementFinderUnitTest {
    @Test
    void findPeakElement_givenArrayOfIntegers_whenPeaksAtExtremes_thenReturnsCorrectPeak() {
        PeakElementFinder finder = new PeakElementFinder();
        int[] array = {5, 2, 1, 3, 4};
        List<Integer> peaks = finder.findPeakElements(array);
        assertEquals(2, peaks.size());
        assertTrue(peaks.contains(5));
        assertTrue(peaks.contains(4));
    }
}
```

### 5.3 处理平台期(连续相等元素)

**如果数组包含连续相等的元素，则返回第一个出现的索引至关重要**，isPeak函数通过跳过连续相等的元素来处理这种情况：

```java
private boolean isPeak(int[] arr, int index, int n) {
    if (index == 0) {
        return n > 1 ? arr[index] >= arr[index + 1] : true;
    } else if (index == n - 1) {
        return arr[index] >= arr[index - 1];
    } else if (arr[index] == arr[index + 1] && arr[index] > arr[index - 1]) {
        int i = index;

        while (i < n - 1 && arr[i] == arr[i + 1]) {
            i++;
        }
        return i == n - 1 || arr[i] > arr[i + 1];
    } else {
        return arr[index] >= arr[index - 1] && arr[index] >= arr[index + 1];
    }
}
```

findPeakElement函数跳过连续相等的元素，确保在识别峰值时返回第一次出现的索引。

```java
public class PeakElementFinderUnitTest {
    @Test
    void findPeakElement_givenArrayOfIntegers_whenPlateaus_thenReturnsCorrectPeak() {
        PeakElementFinder finder = new PeakElementFinder();
        int[] array = {1, 2, 2, 2, 3, 4, 5};
        List<Integer> peaks = finder.findPeakElements(array);
        assertEquals(1, peaks.size());
        assertTrue(peaks.contains(5));
    }
}
```

## 6. 总结

了解查找峰值元素的技术，可以帮助开发人员在设计高效且弹性的算法时做出明智的决策。**发现峰值元素的方法多种多样，每种方法的时间复杂度也各不相同，例如O(log n)或O(n)**。

这些方法的选择取决于具体需求和应用约束，**选择正确的算法与应用中期望实现的效率和性能目标相一致**。