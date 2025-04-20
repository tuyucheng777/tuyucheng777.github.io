---
layout: post
title:  Java中的无状态对象
category: designpattern
copyright: designpattern
excerpt: 无状态对象
---

## 1. 概述

在本教程中，我们将讨论如何在Java中实现无状态对象，**无状态对象是没有实例字段的类的实例**。

在Java中，我们所有的代码都必须放在一个类中。在编写算法时，我们可能只需要在类中提供静态方法来实现这一点。

然而，有时我们希望将我们的算法绑定到无状态对象。

## 2. 对象状态概述

当我们考虑Java中的对象时，我们通常会考虑在字段中包含状态的对象，以及对该状态进行操作以提供行为的方法。

除此之外，我们还可以创建具有不可修改字段的对象，这些对象的状态在创建时就已定义，并且由于其状态不会改变，因此是[不可变](https://www.baeldung.com/java-immutable-object)的。在并发操作中，不可变对象具有与无状态对象相同的优势。

最后，我们有一些对象，它们**要么完全没有字段，要么只有[编译时常量](https://www.baeldung.com/java-compile-time-constants)字段**，这些对象是无状态的。

让我们看看为什么我们可能希望使用无状态对象。

## 3. 使用无状态对象

为了便于示例说明，我们采用一种没有状态的排序算法，我们选择[冒泡排序](https://www.baeldung.com/java-bubble-sort)作为我们的实现：

```java
public void sort(int[] array) {
    int n = array.length;
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < array[j + 1]; j++) {
                int swap = array[j];
                array[j] = array[j + 1];
                array[j + 1] = swap;
            }
        }
    }
}
```

### 3.1 多种无状态排序实现

我们现在想添加使用其他排序算法进行排序的可能性，因此我们考虑[快速排序](https://www.baeldung.com/java-quicksort)算法，它也是无状态的：

```java
public void sort(int[] array) {
    quickSort(array, 0, array.length - 1);
}

private void quickSort(int[] array, int begin, int end) {
    if (begin < end) {
        int pi = partition(array, begin, end);
        quickSort(array, begin, pi - 1);
        quickSort(array, pi + 1, end);
    }
}

private int partition(int[] array, int low, int high) {
    int pivot = array[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (array[j] < pivot) {
            i++;
            int swap = array[i];
            array[i] = array[j];
            array[j] = swap;
        }
    }
    int swap = array[i + 1];
    array[i + 1] = array[high];
    array[high] = swap;
    return i + 1;
}
```

### 3.2 选择实现方式

假设使用哪种算法的决定是在运行时做出的。

我们需要一种在运行时选择正确排序算法的方法，为此，我们使用了一种称为[策略模式](https://www.baeldung.com/java-strategy-pattern)的设计模式。

为了在我们的案例中实现策略模式，我们将创建一个名为SortingStrategy的接口，其中包含sort()方法的签名：

```java
public interface SortingStrategy {   
    void sort(int[] array);
}
```

现在，我们可以将每个排序策略实现为实现此接口的无状态对象。这样，我们可以切换到任何我们喜欢的实现，而我们的消费代码则使用传递给它的任意排序对象：

```java
public class BubbleSort implements SortingStrategy {

    @Override
    public void sort(int[] array) {
        // Bubblesort implementation
    }
}

public class QuickSort implements SortingStrategy {

    @Override
    public void sort(int[] array) {
        // Quicksort implementation
    }
    // Other helpful methods
}
```

这里的类不包含任何字段，因此也没有状态。但是，由于有一个对象，它可以满足我们为所有排序算法定义的通用接口-SortingStrategy。

### 3.3 单例无状态实现

我们希望引入一种方式，让用户能够自行选择排序策略。由于这些类是无状态的，我们不需要创建多个实例。因此，我们可以使用[单例设计模式](https://www.baeldung.com/java-singleton)来实现。

我们可以通过使用[Java枚举](https://www.baeldung.com/a-guide-to-java-enums)来实现策略实例的这种模式。

让我们从class类型切换到enum类型，并添加一个常量INSTANCE，这个常量实际上是该特定排序算法的一个无状态实例。由于枚举可以实现Java接口，因此这是一种提供策略对象单例的简洁方法：

```java
public enum BubbleSort implements SortingStrategy {
    
    INSTANCE;

    @Override
    public void sort(int[] array) {
        // Bubblesort implementation
    }
}

public enum QuickSort implements SortingStrategy {
    
    INSTANCE;

    @Override
    public void sort(int[] array) {
        // Quicksort implementation
    }
    // Other helpful methods
}
```

### 3.4 测试排序策略

最后，我们编写测试以确保两种排序策略均有效并且易于维护：

```java
@Test
void givenArray_whenBubbleSorting_thenSorted() {
    int[] arrayToSort = {17, 6, 11, 41, 5, 3, 4, -9};
    int[] sortedArray = {-9, 3, 4, 5, 6, 11, 17, 41};
        
    SortingStrategy sortingStrategy = BubbleSort.INSTANCE;
    sortingStrategy.sort(arrayToSort);
    assertArrayEquals(sortedArray, arrayToSort);
}
    
@Test
void givenArray_whenQuickSortSorting_thenSorted() {
    int[] arrayToSort = {17, 6, 11, 41, 5, 3, 4, -9};
    int[] sortedArray = {-9, 3, 4, 5, 6, 11, 17, 41};
        
    SortingStrategy sortingStrategy = QuickSort.INSTANCE;
    sortingStrategy.sort(arrayToSort);
    assertArrayEquals(sortedArray, arrayToSort);
}
```

## 4. 总结

在本文中，我们探讨了Java语言中的无状态对象。

我们看到无状态对象对于保存不需要状态的算法很有用，我们还研究了如何实现策略模式。