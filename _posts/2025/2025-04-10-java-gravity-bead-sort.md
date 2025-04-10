---
layout: post
title:  Java中的重力/珠子排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在本教程中，我们将讨论[重力排序](https://www.baeldung.com/cs/gravity-sort)算法及其在Java中的单线程实现。

## 2. 算法

重力排序是一种自然排序算法，其灵感来自自然事件-在本例中是重力。**该算法也称为珠子排序，它模拟重力对正整数列表进行排序**。

**该算法的思路是使用垂直杆和水平杆上的珠子来表示正整数-类似于算盘**，但每个层代表输入列表中的一个数字。下一步是将珠子放到尽可能低的位置，这样算盘上的数字就会按升序排列：

例如，以下是对输入列表[4,2\]进行排序的过程：

![](/assets/images/2025/algorithms/javagravitybeadsort01.png)

**我们将重力作用于算盘，随后，所有珠子在下落后都会处于其可能处于的最低位置，算盘的最终状态现在是从上到下按正整数升序排列**。

实际上，重力会使所有珠子同时落下。然而，在软件中，我们必须模拟珠子在不同迭代中落下的过程。接下来，我们将研究如何将算盘表示为二维数组，以及如何模拟珠子落下的过程来对列表中的数字进行排序。

## 3. 实现

为了在软件中实现重力排序，我们将按照[本文](https://www.baeldung.com/cs/gravity-sort)中的伪代码用Java编写代码。

**首先，我们需要将输入列表转换为算盘**。我们将使用二维数组来表示杆(列)和层(行)作为矩阵的维度，此外，我们将分别使用true或false来表示珠子或空单元格。

**在设置算盘之前，我们先来计算一下矩阵的维数**。列数m等于列表中的最大元素，因此，让我们创建一个方法来查找这个数字：
```java
static int findMax(int[] A) {
    int max = A[0];
    for (int i = 1; i < A.length; i++) {
        if (A[i] > max) {
            max = A[i];
        }
    }
    return max;
}
```

现在，我们可以将最大的数字分配给m：
```java
int[] A = {1, 3, 4, 2};
int m = findMax(A);
```

**有了m，我们现在可以创建算盘的表示**，我们将使用setupAbacus()方法来执行此操作：
```java
static boolean[][] setupAbacus(int[] A, int m) {
    boolean[][] abacus = new boolean[A.length][m];
    for (int i = 0; i < abacus.length; i++) {
        int number = A[i];
        for (int j = 0; j < abacus[0].length && j < number; j++) {
            abacus[A.length - 1 - i][j] = true;
        }
    }
    return abacus;
}
```

**setupAbacus()方法返回算盘的初始状态，该方法遍历矩阵中的每个单元格，分配一个珠子或一个空单元格**。

在矩阵的每一层，我们将使用列表A中的第i个数字来确定一行中珠子的数量。此外，我们遍历每一列j，如果number大于第j列索引，我们将此单元格标记为true以表示珠子。否则，循环提前终止，使第i行中的其余单元格为空或为false。

让我们创建算盘：
```java
boolean[][] abacus = setupAbacus(A, m);
```

**我们现在准备利用重力对珠子进行排序，将它们放到尽可能低的位置**：
```java
static void dropBeads(boolean[][] abacus, int[] A, int m) {
    for (int i = 1; i < A.length; i++) {
        for (int j = m - 1; j >= 0; j--) {
            if (abacus[i][j] == true) {
                int x = i;
                while (x > 0 && abacus[x - 1][j] == false) {
                    boolean temp = abacus[x - 1][j];
                    abacus[x - 1][j] = abacus[x][j];
                    abacus[x][j] = temp;
                    x--;
                }
            }
        }
    }
}
```

最初，dropBeads()方法循环遍历矩阵中的每个单元格。从1开始，i是起始行，因为从最底层0开始不会有任何珠子可以掉落。对于列，我们从j = m – 1开始，从右到左开始掉落珠子。

**在每次迭代中，我们都会检查当前单元格abacus[i\][j\]是否包含珠子。如果包含，我们使用变量x来存储下落珠子的当前层级。只要珠子不是最底层，并且其下方单元格为空，我们就通过减小x来丢弃珠子**。

**最后，我们需要将算盘的最终状态转换为排序数组**，toSortedList()方法将算盘以及原始输入列表作为参数接收，并相应地修改数组：
```java
static void toSortedList(boolean[][] abacus, int[] A) {
    int index = 0;
    for (int i = abacus.length - 1; i >=0; i--) {
        int beads = 0;
        for (int j = 0; j < abacus[0].length && abacus[i][j] == true; j++) {
            beads++;
        }
        A[index++] = beads;
    }
}
```

我们可以回想一下，每行中的珠子数量代表列表中的单个数字，因此，该方法会遍历算盘的每一层，计算珠子数量，并将值分配给列表。**该方法从最高行值开始按升序排列值，但是，从i = 0开始，它将数字按降序排列**。

**我们将算法的所有部分放在一起，放入一个GravitySort()方法中**：
```java
static void gravitySort(int[] A) {
    int m = findMax(A);
    boolean[][] abacus = setupAbacus(A, m);
    dropBeads(abacus, A, m);
    transformToList(abacus, A);
}
```

我们可以通过创建单元测试来确认算法是否有效：
```java
@Test
public void givenIntegerArray_whenSortedWithGravitySort_thenGetSortedArray() {
    int[] actual = {9, 9, 100, 3, 57, 12, 3, 78, 0, 2, 2, 40, 21, 9};
    int[] expected = {0, 2, 2, 3, 3, 9, 9, 9, 12, 21, 40, 57, 78, 100};
    GravitySort.sort(actual);
    Assert.assertArrayEquals(expected, actual);
}
```

## 4. 复杂度分析

我们看到重力排序算法需要大量处理，因此，让我们将其分解为时间和空间复杂度。

### 4.1 时间复杂度

重力排序算法的实现从查找输入列表中的最大数字m开始，这个过程是一个O(n)运行时操作，因为我们遍历数组一次。获取m后，我们设置算盘的初始状态，由于算盘实际上是一个矩阵，因此访问每一行和每一列的每个单元格会导致O(m * n)操作，其中n是行数。

**设置完成后，我们必须将珠子放到矩阵中尽可能低的位置，就像重力影响算盘一样**。同样，我们会访问矩阵中的每个单元格并使用一个内部循环，将珠子在每列中最多放置n层，**这个过程的运行时间为O(n * n * m)**。

此外，我们必须执行O(n)个额外步骤来根据算盘中的排序表示重新创建我们的列表。

总体而言，重力排序是一种O(n * n * m)算法，考虑到它模拟珠子下落的努力。

### 4.2 空间复杂度

重力排序的空间复杂度与输入列表的大小及其最大数量成正比，例如，**一个元素数为n且最大数量为m的列表需要n * m维的矩阵表示。因此，空间复杂度为O(n * m)，用于在内存中为矩阵分配额外的空间**。

尽管如此，我们还是尝试通过用单个位或数字表示珠子和空单元格来最小化空间复杂度。即，1或true表示珠子，同样，0或false值表示空单元格。

## 5. 总结

在本文中，我们学习了如何实现重力排序算法的单线程方法。该算法也称为珠子排序，其灵感来源于重力对算盘中表示为珠子的正整数进行自然排序的现象。然而，在软件中，我们使用二维矩阵和单比特值来重现这种环境。

尽管单线程实现的时间和空间复杂度很高，但该算法在硬件和多线程应用程序中表现良好。尽管如此，重力排序算法仍然体现了自然事件如何启发软件实现的解决方案。