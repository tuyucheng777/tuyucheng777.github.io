---
layout: post
title:  使用Java创建魔方
category: algorithms
copyright: algorithms
excerpt: 魔方
---

## 1. 简介

在本文中，我们将学习如何创建魔方。**我们将了解魔方是什么，创建魔方的算法是什么，以及如何用Java实现它**。

## 2. 什么是魔方？

**魔方是一个数学谜题，我们先从一个大小为n * n的魔方开始，并需要在其中填充数字，使得1到n²之间的每个数字恰好出现一次，并且每行、每列和每对角线的和都等于相同的数字**。

例如，大小为3 * 3的魔方可以是：

![](/assets/images/2025/algorithms/javamagicsquare01.png)

在这里我们可以看到每个单元格都有不同的数字，介于1到9之间；我们还可以看到每行、每列和每对角线的总和为15。

这里恰好还有一项额外的检查，每行、每列、每条对角线的和也等于一个值，只要知道n就可以计算出来。具体来说：

![](/assets/images/2025/algorithms/javamagicsquare02.png)

因此，对于我们的3 * 3魔方，这给出了(3³ + 3) / 2 = 15的值。

事实上，有三种相对简单的算法可用于生成这些，具体取决于魔方的大小：

- 每侧单元格数量为奇数
- 双倍：每边单元格数量为偶数，即每边都是4的倍数
- 单倍：每边单元格数量为偶数，即每边都是2的倍数，但不是4的倍数

我们将研究每个算法以及如何在Java中生成它们。

## 3. 验证魔方

**在生成魔方之前，我们首先需要证明给定的魔方确实满足要求，也就是说每一行、每一列以及每条对角线的和都相同**。

在这样做之前，让我们定义一个类来代表我们的魔方：

```java
public class MagicSquare {
    private int[][] cells;

    public MagicSquare(int n) {
        this.cells = new int[n][];

        for (int i = 0; i < n; ++i) {
            this.cells[i] = new int[n];
        }
    }

    public int getN() {
        return cells.length;
    }

    public int getCell(int x, int y) {
        return cells[x][y];
    }

    public void setCell(int x, int y, int value) {
        cells[x][y] = value;
    }
}
```

这只是一个二维整数数组的包装器，然后，我们确保数组的大小正确，并且可以轻松访问数组中的各个单元。

**现在有了它，让我们编写一个方法来验证该魔方确实是一个魔方**。

首先，我们计算行、列和对角线的预期值总和为：

```java
int n = getN();
int expectedValue = ((n * n * n) + n) / 2;
```

接下来，我们将开始对数值求和，并检查结果是否符合预期。我们先计算对角线：

```java
// Diagonals
if (IntStream.range(0, n).map(i -> getCell(i, i)).sum() != expectedValue) {
    throw new IllegalStateException("Leading diagonal is not the expected value");
}
if (IntStream.range(0, n).map(i -> getCell(i, n - i - 1)).sum() != expectedValue) {
    throw new IllegalStateException("Trailing diagonal is not the expected value");
}
```

这只是从0到n的迭代，取该点上适当对角线上的每个单元格并将它们相加。

接下来是行和列：

```java
// Rows
IntStream.range(0, n).forEach(y -> {
    if (IntStream.range(0, n).map(x -> getCell(x, y)).sum() != expectedValue) {
        throw new IllegalStateException("Row is not the expected value");
    }
});

// Cols
IntStream.range(0, n).forEach(x -> {
    if (IntStream.range(0, n).map(y -> getCell(x, y)).sum() != expectedValue) {
        throw new IllegalStateException("Column is not the expected value");
    }
});
```

这里，我们根据需要迭代每行或每列的所有单元格，并计算所有单元格值的和。如果在任何一种情况下，我们得到的值与预期不同，那么我们将抛出异常，表明出现了问题。

## 4. 生成魔方

**至此，我们可以正确验证任何给定的方格是否是魔方，现在我们需要能够生成它们**。

我们之前看到过，根据魔方的大小，这里有三种不同的算法可以使用，我们将依次介绍每种算法。

### 4.1 奇数魔方的算法

**我们将要研究的第一个算法是针对每边具有奇数个单元格的魔方**。

当生成这种大小的魔方时，我们总是先把第一个数字放在最上面一行的中间格子里。然后，我们依次按如下方式放置后续的数字：

- 首先，我们尝试将其放置在上一个方格正上方的单元格中。在执行此操作时，我们会沿边缘进行环绕，例如，在顶行之后，我们会移动到底行。
- 如果该单元格已填充，则将下一个数字放在紧邻前一个数字的下方单元格中。同样，在执行此操作时，我们会沿边缘进行环绕。

例如，在我们的3 * 3魔方中，我们从顶部中间的单元格开始。然后我们向上和向右移动，绕一圈找到右下角的单元格：

![](/assets/images/2025/algorithms/javamagicsquare03.png)

从这里开始，我们先向上右移动，填充中间左侧的单元格。之后，再向上右移动，又回到了第一个单元格，所以我们必须向下移动到左下角的单元格：

![](/assets/images/2025/algorithms/javamagicsquare04.png)

如果我们继续这样做，我们最终会用有效的魔方填满每个方格。

### 4.2 奇数魔方的实现

那么我们怎样在Java中做到这一点？

首先，我们来输入第一个数字，它位于最上面一行的中间单元格：

```java
int y = 0;
int x = (n - 1) / 2;
setCell(x, y, 1);
```

完成此操作后，我们将循环遍历所有其他数字，并依次放置每个数字：

```java
for (int number = 2; number <= n * n; ++number) {
    int nextX = ...;
    int nextY = ...;

    setCell(nextX, nextY, number);

    x = nextX;
    y = nextY;
}
```

现在我们只需要确定nextX和nextY使用什么值。

首先，我们将尝试向上和向右移动，并根据需要环绕：

```java
int nextX = x + 1;
if (nextX == n) {
    nextX = 0;
}

int nextY = y - 1;
if (nextY == -1) {
    nextY = n - 1;
}
```

然而，我们还需要处理下一个单元格已经被占用的情况：

```java
if (getCell(nextX, nextY) != 0) {
    nextX = x;

    nextY = y + 1;
    if (nextY == n) {
        nextY = 0;
    }
}
```

**综上所述，我们实现了生成任意奇数大小的魔方**。例如，用这个生成的9 * 9魔方如下：

![](/assets/images/2025/algorithms/javamagicsquare05.png)

### 4.3 双偶数魔方算法

**上述算法适用于奇数大小的魔方，但不适用于偶数大小的魔方**。事实上，对于偶数大小的魔方，我们需要根据具体大小选择两种算法之一。

**双偶数魔方是指边长为4的倍数的魔方，例如4 * 4、8 * 8、12 * 12等**。为了生成这些魔方，我们需要在魔方中划分出4个特殊区域，这些区域是距离每条边n/4且距离角点n/4以上的格子：

![](/assets/images/2025/algorithms/javamagicsquare06.png)

完成这些后，我们现在填充数字。这需要两遍，第一遍从左上角开始，从左到右逐行进行，每当我们到达一个未突出显示的单元格时，我们都会按以下顺序添加下一个数字：

![](/assets/images/2025/algorithms/javamagicsquare07.png)

第二遍完全相同，但是从右下角开始，从右到左运行，并且仅向突出显示的单元格添加数字：

![](/assets/images/2025/algorithms/javamagicsquare08.png)

此时，我们得到了有效的魔方。

### 4.4 双偶数魔方的实现

为了在Java中实现这一点，我们需要利用两种技术。

**首先，我们实际上可以一次性将所有数字相加，如果我们在未高亮的单元格上，则操作与之前相同；但如果我们在高亮的单元格上，则改为从n²开始倒数**：

```java
int number = 1;

for (int y = 0; y < n; ++y) {
    for (int x = 0; x < n; ++x) {
        boolean highlighted = ...;
        
        if (highlighted) {
            setCell(x, y, (n * n) - number + 1);
        } else {
            setCell(x, y, number);
        }

        number += 1;
    }
}
```

**现在我们只需要确定高亮的方块在哪里，我们可以通过检查x和y坐标是否在范围内来做到这一点**：

```java
if ((y < n/4 || y >= 3*n/4) && (x >= n/4 && x < 3*n/4)) {
    highlighted = true;
} else if ((x < n/4 || x >= 3*n/4) && (y >= n/4 && y < 3*n/4)) {
    highlighted = true;
}
```

第一个条件是针对魔方顶部和底部的高亮区域，而第二个条件是针对魔方左侧和右侧的高亮区域。我们可以看到它们实际上是相同的，只是在检查中x和y轴被调换了位置。在这两种情况下，我们的位置都在魔方该侧1/4以内，并且距离相邻角1/4到3/4之间。

**综上所述，我们实现了生成任意双偶数大小的魔方**。例如，我们用这个生成的8 * 8魔方如下：

![](/assets/images/2025/algorithms/javamagicsquare09.png)

### 4.5 单偶数魔方的算法

**我们最终的魔方尺寸是单偶方格，也就是说，边长能被2整除，但不能被4整除。这也要求边长至少为6个单元-2 * 2的魔方没有解，所以6 * 6是我们能解的最小的单偶魔方**。

我们首先将它们分成四等份-每等份都是奇数大小的魔方，然后使用与奇数大小魔方相同的算法进行填充，只是为每个象限分配不同的数字范围-从左上角的四分之一开始，然后是右下角、右上角，最后是左下角：

![](/assets/images/2025/algorithms/javamagicsquare10.png)

**即使我们用奇数大小算法填充了所有方块，我们仍然没有得到一个有效的魔方**。此时我们会注意到，所有列的和都是正确的，但行和对角线的和却不正确。然而，我们也会注意到，上半部分的行和都相同，下半部分的行和也都相同：

![](/assets/images/2025/algorithms/javamagicsquare11.png)

我们可以通过在上半部分和下半部分之间进行多次交换来解决这个问题，如下所示：

- 位于左上象限顶行、中心左侧的每个单元格
- 位于左上象限底部行、中心左侧的每个单元格
- 这些单元格之间每行的单元格数量相同，但从一个单元格开始
- 右上象限中每行的单元格比这些少一个，但从右侧开始

如下所示：

![](/assets/images/2025/algorithms/javamagicsquare12.png)

**现在这是一个有效的魔方，每一行、每一列和每条对角线加起来都相同-在本例中为505**。

### 4.6 单偶数魔方的实现

**这个实现将基于我们对奇数大小魔方的操作**，首先，我们需要计算一些用于生成的值：

```java
int halfN = n/2;
int swapSize = n/4;
```

注意，我们计算swapSize时，将其设为n / 4，然后将其存入一个int类型。这实际上是向下取整，所以当n = 10时，我们得到的swapSize值为2。

接下来，我们需要填充网格。假设我们已经有一个函数，用于执行奇数大小的魔方算法，并且只进行适当的偏移：

```java
populateOddArea(0,     0,     halfN, 0);
populateOddArea(halfN, halfN, halfN, halfN * halfN);
populateOddArea(halfN, 0,     halfN, (halfN * halfN) * 2);
populateOddArea(0,     halfN, halfN, (halfN * halfN) * 3);
```

现在我们只需要执行交换操作，同样，我们假设我们有一个函数可以交换方块中的单元格。

交换左象限中的单元格只需从左侧迭代swapSize并执行交换即可：

```java
for (int x = 0; x < swapSize; ++x) {
    swapCells(x, 0, x, halfN); // Top row
    swapCells(x, halfN - 1, x, n - 1); // Bottom row
    
    // All in-between rows.
    for (int y = 1; y < halfN - 1; ++y) {
        swapCells(x + 1, y, x + 1, y + halfN);
    }
}
```

最后，我们交换右象限的单元格，具体方法是从右侧开始迭代swapSize - 1并执行交换操作：

```java
for (int x = 0; x < swapSize - 1; ++x) {
    for (int y = 0; y < halfN; ++y) {
        swapCells(n - x - 1, y, n - x - 1, y + halfN);
    }
}
```

**综上所述，我们实现了生成任意单偶数大小的魔方**。例如，我们用这个方法生成的10 * 10魔方如下：

![](/assets/images/2025/algorithms/javamagicsquare13.png)

## 5. 总结

本文，我们研究了用于创建魔方的算法，并了解了如何在Java中实现这些算法。