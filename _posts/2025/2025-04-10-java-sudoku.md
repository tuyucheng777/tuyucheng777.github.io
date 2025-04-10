---
layout: post
title:  使用Java创建数独求解器
category: algorithms
copyright: algorithms
excerpt: 数独求解器
---

## 1. 概述

在本文中，我们将研究数独谜题及其所使用的算法。

接下来，我们将用Java实现解决方案。第一个解决方案将是简单的暴力破解，第二个解决方案将利用[Dancing Links](https://en.wikipedia.org/wiki/Dancing_Links)技术。

记住，我们的重点是算法而不是OOP设计。

## 2. 数独谜题

简而言之，数独是一种组合数字放置谜题，其中9 x 9网格部分填充了从1到9的数字，目标是用剩余的数字填充剩余的空白字段，使得每行和每列都只有一个数字。

此外，网格的每个3 x 3子区域也不能有重复的数字。随着每个棋盘上空白区域数量的增加，难度自然也会随之提升。

### 2.1 测试板

为了使我们的解决方案更有趣并验证算法，我们将使用[世界上最难的数独](http://www.telegraph.co.uk/news/science/science-news/9359579/Worlds-hardest-sudoku-can-you-crack-it.html)板，即：
```text
8 . . . . . . . . 
. . 3 6 . . . . . 
. 7 . . 9 . 2 . . 
. 5 . . . 7 . . . 
. . . . 4 5 7 . . 
. . . 1 . . . 3 . 
. . 1 . . . . 6 8 
. . 8 5 . . . 1 . 
. 9 . . . . 4 . .
```

### 2.2 已解决的棋盘

并且，快速地揭示解决方案-正确解决的谜题将给我们以下结果：
```text
8 1 2 7 5 3 6 4 9 
9 4 3 6 8 2 1 7 5 
6 7 5 4 9 1 2 8 3 
1 5 4 2 3 7 8 9 6 
3 6 9 8 4 5 7 2 1 
2 8 7 1 6 9 5 3 4 
5 2 1 9 7 4 3 6 8 
4 3 8 5 2 6 9 1 7 
7 9 6 3 1 8 4 5 2
```

## 3. 回溯算法

### 3.1 介绍

**[回溯算法](https://www.baeldung.com/cs/backtracking-algorithms)尝试通过测试每个单元格来寻找有效的解决方案来解决难题**。

如果没有违反约束，算法将移动到下一个单元格，填写所有潜在解决方案并重复所有检查。

如果存在违规，则算法会递增单元格的值，一旦单元格的值达到9，并且仍然有违规，算法就会返回到前一个单元格并增加该单元格的值。

它尝试所有可能的解决方案。

### 3.2 解决方案

首先，我们将棋盘定义为一个二维整数数组，我们将使用0作为空单元格。
```java
int[][] board = {
    { 8, 0, 0, 0, 0, 0, 0, 0, 0 },
    { 0, 0, 3, 6, 0, 0, 0, 0, 0 },
    { 0, 7, 0, 0, 9, 0, 2, 0, 0 },
    { 0, 5, 0, 0, 0, 7, 0, 0, 0 },
    { 0, 0, 0, 0, 4, 5, 7, 0, 0 },
    { 0, 0, 0, 1, 0, 0, 0, 3, 0 },
    { 0, 0, 1, 0, 0, 0, 0, 6, 8 },
    { 0, 0, 8, 5, 0, 0, 0, 1, 0 },
    { 0, 9, 0, 0, 0, 0, 4, 0, 0 } 
};
```

让我们创建一个solve()方法，以board作为输入参数并遍历行、列和值，测试每个单元格是否有有效的解决方案：
```java
private boolean solve(int[][] board) {
   for (int row = BOARD_START_INDEX; row < BOARD_SIZE; row++) {
      for (int column = BOARD_START_INDEX; column < BOARD_SIZE; column++) {
         if (board[row][column] == NO_VALUE) {
            for (int k = MIN_VALUE; k <= MAX_VALUE; k++) {
               board[row][column] = k;
               if (isValid(board, row, column) && solve(board)) {
                  return true;
               }
               board[row][column] = NO_VALUE;
            }
            return false;
         }
      }
   }
   return true;
}
```

我们需要的另一种方法是isValid()方法，它将检查数独约束，即检查行、列和3 x 3网格是否有效：
```java
private boolean isValid(int[][] board, int row, int column) {
   return (rowConstraint(board, row)
           && columnConstraint(board, column)
           && subsectionConstraint(board, row, column));
}
```

这3个检查比较类似，首先我们先来看一下行检查：
```java
private boolean rowConstraint(int[][] board, int row) {
   boolean[] constraint = new boolean[BOARD_SIZE];
   for (int column = BOARD_START_INDEX; column < BOARD_SIZE; column++) {
      if (!checkConstraint(board, row, constraint, column)) {
         return false;
      }
   }
   return true;
}
```

接下来，我们使用几乎相同的代码来验证列：
```java
private boolean columnConstraint(int[][] board, int column) {
   boolean[] constraint = new boolean[BOARD_SIZE];
   for (int row = BOARD_START_INDEX; row < BOARD_SIZE; row++) {
      if (!checkConstraint(board, row, constraint, column)) {
         return false;
      }
   }
   return true;
}
```

此外，我们还需要验证3 x 3子部分：
```java
private boolean subsectionConstraint(int[][] board, int row, int column) {
   boolean[] constraint = new boolean[BOARD_SIZE];
   int subsectionRowStart = (row / SUBSECTION_SIZE) * SUBSECTION_SIZE;
   int subsectionRowEnd = subsectionRowStart + SUBSECTION_SIZE;

   int subsectionColumnStart = (column / SUBSECTION_SIZE) * SUBSECTION_SIZE;
   int subsectionColumnEnd = subsectionColumnStart + SUBSECTION_SIZE;

   for (int r = subsectionRowStart; r < subsectionRowEnd; r++) {
      for (int c = subsectionColumnStart; c < subsectionColumnEnd; c++) {
         if (!checkConstraint(board, r, constraint, c)) return false;
      }
   }
   return true;
}
```

最后，我们需要一个checkConstraint()方法：
```java
boolean checkConstraint(
        int[][] board,
        int row,
        boolean[] constraint,
        int column) {
   if (board[row][column] != NO_VALUE) {
      if (!constraint[board[row][column] - 1]) {
         constraint[board[row][column] - 1] = true;
      } else {
         return false;
      }
   }
   return true;
}
```

一旦完成，isValid()方法就可以简单地返回true。

现在我们几乎可以测试解决方案了，算法已经完成；但是，它仅返回true或false。

因此，为了直观地检查棋盘，我们只需要打印出结果即可。显然，这不是算法的一部分。
```java
private void printBoard() {
   for (int row = BOARD_START_INDEX; row < BOARD_SIZE; row++) {
      for (int column = BOARD_START_INDEX; column < BOARD_SIZE; column++) {
         System.out.print(board[row][column] + " ");
      }
      System.out.println();
   }
}
```

我们已经成功实现了解决数独难题的回溯算法！

显然，还有改进的空间，因为该算法会一遍又一遍地检查每种可能的组合(即使我们知道特定的解决方案是无效的)。

## 4. Dancing Links

### 4.1 精确覆盖

我们来看另一种解决方案，数独可以描述为一个[精确覆盖](https://en.wikipedia.org/wiki/Exact_cover)问题，可以用关联矩阵来表示两个对象之间的关系。

例如，如果我们取从1到7的数字和集合S = {A,B,C,D,E,F}，其中：

- A = {1,4,7}
- B = {1,4}
- C = {4,5,7}
- D = {3,5,6}
- E = {2,3,6,7}
- F = {2,7}

我们的目标是选择这样的子集，使得每个数字只出现一次并且没有重复，因此得名。

我们可以使用矩阵来表示该问题，其中列是数字，行是集合：
```text
  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 
A | 1 | 0 | 0 | 1 | 0 | 0 | 1 |
B | 1 | 0 | 0 | 1 | 0 | 0 | 0 |
C | 0 | 0 | 0 | 1 | 1 | 0 | 1 |
D | 0 | 0 | 1 | 0 | 1 | 1 | 0 |
E | 0 | 1 | 1 | 0 | 0 | 1 | 1 |
F | 0 | 1 | 0 | 0 | 0 | 0 | 1 |
```

子集合S<sup>*</sup> = {B,D,F}是精确覆盖：
```text
  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 
B | 1 | 0 | 0 | 1 | 0 | 0 | 0 |
D | 0 | 0 | 1 | 0 | 1 | 1 | 0 |
F | 0 | 1 | 0 | 0 | 0 | 0 | 1 |
```

在所有选定的行中，每列恰好有一个1。

### 4.2 算法X

算法X是一种“反复试验的方法”，用于找到精确覆盖问题的所有解决方案，即从我们的示例集合S = {A,B,C,D,E,F}开始，找到子集合S<sup>*</sup> = {B,D,F}。

算法X的工作原理如下：

1. 如果矩阵A没有列，则当前部分解为有效解；成功终止，否则选择列c(确定性)
2. 选择一行r使得A<sub>r,c</sub> = 1(非确定性，即尝试所有可能性)
3. 将行r纳入部分解中
4. 对于满足A<sub>r,c</sub> = 1的每列j，对于满足A<sub>i,j</sub> = 1的每行i，从矩阵A中删除行i，并从矩阵A中删除列j
5. 在约化矩阵A上递归重复此算法

算法X的一个有效实现是Donald Knuth博士提出的[Dancing Links](https://www.ocf.berkeley.edu/~jchu/publicportal/sudoku/0011047.pdf)算法(简称DLX)。

以下解决方案深受[此](https://github.com/rafalio/dancing-links-java)Java实现的启发。

### 4.3 精确覆盖问题

首先，我们需要创建一个矩阵，将数独谜题表示为精确覆盖问题。该矩阵将有9^3行，即每个可能数字(9个数字)的每个可能位置(9行 x 9列)对应一行。

列将代表板(同样是9 x 9)乘以约束的数量。

我们已经定义了3个约束：

- 每行每种只有一个数字
- 每列每种只有一个数字
- 每个小节每种类型只有一个数字

此外，还有隐含的第四个约束：

- 每个单元格中只能有一个数字

这总共给出了4个约束，因此精确覆盖矩阵中有9 x 9 x 4列：
```java
private static int BOARD_SIZE = 9;
private static int SUBSECTION_SIZE = 3;
private static int NO_VALUE = 0;
private static int CONSTRAINTS = 4;
private static int MIN_VALUE = 1;
private static int MAX_VALUE = 9;
private static int COVER_START_INDEX = 1;
```

```java
private int getIndex(int row, int column, int num) {
    return (row - 1) * BOARD_SIZE * BOARD_SIZE 
      + (column - 1) * BOARD_SIZE + (num - 1);
}
```

```java
private boolean[][] createExactCoverBoard() {
    boolean[][] coverBoard = new boolean
      [BOARD_SIZE * BOARD_SIZE * MAX_VALUE]
      [BOARD_SIZE * BOARD_SIZE * CONSTRAINTS];

    int hBase = 0;
    hBase = checkCellConstraint(coverBoard, hBase);
    hBase = checkRowConstraint(coverBoard, hBase);
    hBase = checkColumnConstraint(coverBoard, hBase);
    checkSubsectionConstraint(coverBoard, hBase);
    
    return coverBoard;
}

private int checkSubsectionConstraint(boolean[][] coverBoard, int hBase) {
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row += SUBSECTION_SIZE) {
        for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column += SUBSECTION_SIZE) {
            for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++, hBase++) {
                for (int rowDelta = 0; rowDelta < SUBSECTION_SIZE; rowDelta++) {
                    for (int columnDelta = 0; columnDelta < SUBSECTION_SIZE; columnDelta++) {
                        int index = getIndex(row + rowDelta, column + columnDelta, n);
                        coverBoard[index][hBase] = true;
                    }
                }
            }
        }
    }
    return hBase;
}

private int checkColumnConstraint(boolean[][] coverBoard, int hBase) {
    for (int column = COVER_START_INDEX; column <= BOARD_SIZE; c++) {
        for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++, hBase++) {
            for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row++) {
                int index = getIndex(row, column, n);
                coverBoard[index][hBase] = true;
            }
        }
    }
    return hBase;
}

private int checkRowConstraint(boolean[][] coverBoard, int hBase) {
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; r++) {
        for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++, hBase++) {
            for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column++) {
                int index = getIndex(row, column, n);
                coverBoard[index][hBase] = true;
            }
        }
    }
    return hBase;
}

private int checkCellConstraint(boolean[][] coverBoard, int hBase) {
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row++) {
        for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column++, hBase++) {
            for (int n = COVER_START_INDEX; n <= BOARD_SIZE; n++) {
                int index = getIndex(row, column, n);
                coverBoard[index][hBase] = true;
            }
        }
    }
    return hBase;
}
```

接下来，我们需要使用初始拼图布局更新新创建的棋盘：

```java
private boolean[][] initializeExactCoverBoard(int[][] board) {
    boolean[][] coverBoard = createExactCoverBoard();
    for (int row = COVER_START_INDEX; row <= BOARD_SIZE; row++) {
        for (int column = COVER_START_INDEX; column <= BOARD_SIZE; column++) {
            int n = board[row - 1][column - 1];
            if (n != NO_VALUE) {
                for (int num = MIN_VALUE; num <= MAX_VALUE; num++) {
                    if (num != n) {
                        Arrays.fill(coverBoard[getIndex(row, column, num)], false);
                    }
                }
            }
        }
    }
    return coverBoard;
}
```

现在我们准备进入下一阶段，让我们创建两个类来将单元格连接在一起。

### 4.4 Dancing Links

Dancing Links算法基于以下基本观察，即对节点的双向链表进行以下操作：
```text
node.prev.next = node.next
node.next.prev = node.prev
```

删除节点，同时：
```text
node.prev = node
node.next = node
```

恢复节点。

DLX中的每个节点都与左边、右边、上边和下边的节点相连。

DancingNode类将具有添加或删除节点所需的所有操作：
```java
class DancingNode {
   DancingNode L, R, U, D;
   ColumnNode C;

   DancingNode hookDown(DancingNode node) {
      assert (this.C == node.C);
      node.D = this.D;
      node.D.U = node;
      node.U = this;
      this.D = node;
      return node;
   }

   DancingNode hookRight(DancingNode node) {
      node.R = this.R;
      node.R.L = node;
      node.L = this;
      this.R = node;
      return node;
   }

   void unlinkLR() {
      this.L.R = this.R;
      this.R.L = this.L;
   }

   void relinkLR() {
      this.L.R = this.R.L = this;
   }

   void unlinkUD() {
      this.U.D = this.D;
      this.D.U = this.U;
   }

   void relinkUD() {
      this.U.D = this.D.U = this;
   }

   DancingNode() {
      L = R = U = D = this;
   }

   DancingNode(ColumnNode c) {
      this();
      C = c;
   }
}
```

### 4.5 列节点

ColumnNode类将列链接在一起：
```java
class ColumnNode extends DancingNode {
   int size;
   String name;

   ColumnNode(String n) {
      super();
      size = 0;
      name = n;
      C = this;
   }

   void cover() {
      unlinkLR();
      for (DancingNode i = this.D; i != this; i = i.D) {
         for (DancingNode j = i.R; j != i; j = j.R) {
            j.unlinkUD();
            j.C.size--;
         }
      }
   }

   void uncover() {
      for (DancingNode i = this.U; i != this; i = i.U) {
         for (DancingNode j = i.L; j != i; j = j.L) {
            j.C.size++;
            j.relinkUD();
         }
      }
      relinkLR();
   }
}
```

### 4.6 求解器

接下来，我们需要创建一个由DancingNode和ColumnNode对象组成的网格：
```java
private ColumnNode makeDLXBoard(boolean[][] grid) {
   int COLS = grid[0].length;

   ColumnNode headerNode = new ColumnNode("header");
   List<ColumnNode> columnNodes = new ArrayList<>();

   for (int i = 0; i < COLS; i++) {
      ColumnNode n = new ColumnNode(Integer.toString(i));
      columnNodes.add(n);
      headerNode = (ColumnNode) headerNode.hookRight(n);
   }
   headerNode = headerNode.R.C;

   for (boolean[] aGrid : grid) {
      DancingNode prev = null;
      for (int j = 0; j < COLS; j++) {
         if (aGrid[j]) {
            ColumnNode col = columnNodes.get(j);
            DancingNode newNode = new DancingNode(col);
            if (prev == null) prev = newNode;
            col.U.hookDown(newNode);
            prev = prev.hookRight(newNode);
            col.size++;
         }
      }
   }

   headerNode.size = COLS;

   return headerNode;
}
```

我们将使用启发式搜索来查找列并返回矩阵的子集：
```java
private ColumnNode selectColumnNodeHeuristic() {
   int min = Integer.MAX_VALUE;
   ColumnNode ret = null;
   for (ColumnNode c = (ColumnNode) header.R; c != header; c = (ColumnNode) c.R) {
      if (c.size < min) {
         min = c.size;
         ret = c;
      }
   }
   return ret;
}
```

最后，我们可以递归地寻找答案：
```java
private void search(int k) {
   if (header.R == header) {
      handleSolution(answer);
   } else {
      ColumnNode c = selectColumnNodeHeuristic();
      c.cover();

      for (DancingNode r = c.D; r != c; r = r.D) {
         answer.add(r);

         for (DancingNode j = r.R; j != r; j = j.R) {
            j.C.cover();
         }

         search(k + 1);

         r = answer.remove(answer.size() - 1);
         c = r.C;

         for (DancingNode j = r.L; j != r; j = j.L) {
            j.C.uncover();
         }
      }
      c.uncover();
   }
}
```

如果没有更多的列，那么我们可以打印出已解决的数独板。

## 5. 基准测试

我们可以在同一台计算机上运行这两种不同的算法来进行比较(这样可以避免组件、CPU或RAM速度等的差异)，实际时间会因计算机而异。

但是，我们应该能够看到相对结果，这将告诉我们哪种算法运行得更快。

回溯算法大约需要250毫秒来解决棋盘问题。

如果我们将其与大约需要50毫秒的Dancing Links进行比较，可以看到明显的赢家。在解决这个特定示例时，Dancing Links的速度大约快5倍。

## 6. 总结

在本教程中，我们讨论了使用核心Java解决数独难题的两种方法。回溯算法是一种强力算法，可以轻松解决标准9 × 9难题。

我们还讨论了稍微复杂一些的Dancing Links算法，两者都可以在几秒钟内解决最难的难题。