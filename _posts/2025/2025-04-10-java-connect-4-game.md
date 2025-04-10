---
layout: post
title:  用Java实现四子棋游戏
category: algorithms
copyright: algorithms
excerpt: 四子棋游戏
---

## 1. 简介

**在本文中，我们将了解如何使用Java实现[Connect 4](https://en.wikipedia.org/wiki/Connect_Four)游戏**，我们将了解游戏的外观和玩法，然后研究如何实现这些规则。

## 2. 什么是Connect 4？

在实现游戏之前，我们需要了解游戏规则。

四子棋是一款相对简单的游戏，玩家轮流将棋子放到一组棋堆的顶部，每轮结束后，如果任何玩家的棋子在任何直线方向(水平、垂直或对角线)形成一条四条线，则该玩家获胜：

![](/assets/images/2025/algorithms/javaconnect4game01.png)

如果没有，则由下一位玩家代替，如此重复，直到一位玩家获胜或游戏无法获胜。

值得注意的是，玩家可以自由选择将棋子放在哪一列，但该棋子必须放在牌堆顶部，他们无法自由选择将棋子放在列中的哪一行。

**要将其构建为计算机游戏，我们需要考虑几个不同的组件：游戏板本身、玩家放置令牌的能力以及检查游戏是否获胜的能力**，我们将依次介绍这些内容。

## 3. 定义游戏板

**在玩游戏之前，我们首先需要一个游戏空间，这就是游戏板，它包含玩家可以下棋的所有格子，并指示玩家已经放置棋子的位置**。

我们首先编写一个枚举来表示玩家可以在游戏中使用的部件：
```java
public enum Piece {
    PLAYER_1,
    PLAYER_2
}
```

这假设游戏中只有两个玩家，这是四子棋的典型特征。

现在，我们将创建一个代表游戏板的类：
```java
public class GameBoard {
    private final List<List<Piece>> columns;

    private final int rows;

    public GameBoard(int columns, int rows) {
        this.rows = rows;
        this.columns = new ArrayList<>();

        for (int i = 0; i < columns; ++i) {
            this.columns.add(new ArrayList<>());
        }
    }

    public int getRows() {
        return rows;
    }

    public int getColumns() {
        return columns.size();
    }
}
```

这里，我们用一个列表来表示游戏板，每个列表代表游戏中的一整列，列表中的每个条目都表示该列中的一个棋子。

**部件必须从底部堆叠，因此我们不需要考虑间隙。相反，所有间隙都在插入部件上方的列顶部。因此，我们实际上是按照部件添加到列的顺序来存储它们**。

接下来，我们将添加一个助手来获取当前位于棋盘上任何给定单元格中的棋子：
```java
public Piece getCell(int x, int y) {
    assert(x >= 0 && x < getColumns());
    assert(y >= 0 && y < getRows());

    List<Piece> column = columns.get(x);

    if (column.size() > y) {
        return column.get(y);
    } else {
        return null;
    }
}
```

该方法需要从第一列开始的X坐标和从最后一行开始的Y坐标，然后，我们将返回该单元格的正确Piece，如果该单元格中尚无任何内容，则返回null。

## 4. 游戏动作

**现在我们有了游戏板，我们需要能够在上面移动棋子，玩家通过将棋子添加到给定列的顶部来移动**。因此，我们只需添加一个新方法来实现，该方法接收给定列和移动棋子的玩家：
```java
public void move(int x, Piece player) {
    assert(x >= 0 && x < getColumns());

    List<Piece> column = columns.get(x);

    if (column.size() >= this.rows) {
        throw new IllegalArgumentException("That column is full");
    }

    column.add(player);
}
```

我们还在这里添加了额外的检查，如果相关列中已经有太多棋子，则会抛出异常，而不允许玩家移动。

## 5. 检查获胜条件

**玩家移动后，下一步是检查他们是否获胜，这意味着在棋盘上寻找同一玩家在水平、垂直或对角线上的4个棋子**。

但是，我们可以做得更好。根据游戏玩法，我们可以了解一些事实，从而简化搜索过程。

首先，由于游戏在获胜一步后结束，因此只有刚刚移动的玩家才能获胜，这意味着我们只需要检查该玩家棋子的走线。

其次，获胜线必须包含刚刚放置的棋子，这意味着我们不需要搜索整个棋盘，而只需搜索包含所下棋子的子集。

第三，由于游戏的列式结构，我们可以忽略某些不可能的情况。例如，只有当最新棋子至少在第4行时，我们才能形成一条垂直线。如果棋子位于第4行以下，就不能形成一条垂直线。

最终，这意味着我们要搜索以下集合：

- 从最新一块开始，向下延伸三行的一条垂直线
- 四条可能的水平线之一：第一条从左侧三列开始，到我们最新的部分结束，而最后一条从我们最新的部分开始，到右侧三列结束
- 四条可能的前向对角线之一：第一条从最新棋子左侧三列、上方三行开始，而最后一条从最新棋子开始，到右侧三列、下方三行结束
- 四条可能的尾对角线之一：第一条从最新棋子左侧三列、下方三行开始，而最后一条从最新棋子开始，到右侧三列、上方三行结束

**这意味着每次移动后，我们必须检查最多13条可能的路线-并且由于棋盘的大小，其中一些路线可能是不可能的**：

![](/assets/images/2025/algorithms/javaconnect4game02.png)

例如，在这里我们可以看到有几条线超出了游戏区域，因此永远不可能成为获胜线。

### 5.1 检查获胜线

**我们首先需要的是一个检查给定线段的方法，该方法会获取线段的起点和方向，并检查该线上的每个单元格是否都属于当前玩家**：
```java
private boolean checkLine(int x1, int y1, int xDiff, int yDiff, Piece player) {
    for (int i = 0; i < 4; ++i) {
        int x = x1 + (xDiff * i);
        int y = y1 + (yDiff * i);

        if (x < 0 || x > columns.size() - 1) {
            return false;
        }

        if (y < 0 || y > rows - 1) {
            return false;
        }

        if (player != getCell(x, y)) {
            return false;
        }
    }

    return true;
}
```

我们还会检查单元格是否存在，如果检查到单元格不存在，就立即返回“这不是一条必胜线”。我们可以在循环之前这样做，但我们只检查4个单元格，而计算出线的起始和结束位置所带来的额外复杂性在本例中并没有好处。

### 5.2 检查所有可能的线路

**接下来，我们需要检查所有可能的线，如果其中任何一条返回true，那么我们可以立即停止并宣布玩家获胜**。毕竟，即使他们在同一步中获得了多条获胜路线，也并不重要：
```java
private boolean checkWin(int x, int y, Piece player) {
    // Vertical line
    if (checkLine(x, y, 0, -1, player)) {
        return true;
    }

    for (int offset = 0; offset < 4; ++offset) {
        // Horizontal line
        if (checkLine(x - 3 + offset, y, 1, 0, player)) {
            return true;
        }

        // Leading diagonal
        if (checkLine(x - 3 + offset, y + 3 - offset, 1, -1, player)) {
            return true;
        }

        // Trailing diagonal
        if (checkLine(x - 3 + offset, y - 3 + offset, 1, 1, player)) {
            return true;
        }
    }

    return false;
}
```

它使用从左到右的滑动偏移量，并以此确定每行的起始位置。每行从向左滑动3个单元格开始，因为第4个单元格是我们当前正在玩的单元格，必须包含它。最后检查的行从刚刚玩到的单元格开始，向右滑动3个单元格。

最后，我们更新move()函数来检查获胜状态并相应地返回true或false：
```java
public boolean move(int x, Piece player) {
    // Unchanged from before.

    return checkWin(x, column.size() - 1, player);
}
```

### 5.3 玩游戏

至此，我们已经有了一个可玩的游戏，我们可以创建一个新的游戏板并轮流放置棋子，直到我们获得胜利：
```java
GameBoard gameBoard = new GameBoard(8, 6);

assertFalse(gameBoard.move(3, Piece.PLAYER_1));
assertFalse(gameBoard.move(2, Piece.PLAYER_2));

assertFalse(gameBoard.move(4, Piece.PLAYER_1));
assertFalse(gameBoard.move(3, Piece.PLAYER_2));

assertFalse(gameBoard.move(5, Piece.PLAYER_1));
assertFalse(gameBoard.move(6, Piece.PLAYER_2));

assertFalse(gameBoard.move(5, Piece.PLAYER_1));
assertFalse(gameBoard.move(4, Piece.PLAYER_2));

assertFalse(gameBoard.move(5, Piece.PLAYER_1));
assertFalse(gameBoard.move(5, Piece.PLAYER_2));

assertFalse(gameBoard.move(6, Piece.PLAYER_1));
assertTrue(gameBoard.move(4, Piece.PLAYER_2));
```

这组动作正是我们在一开始看到的，我们可以看到最后的活动如何返回，现在已经赢得了比赛。

## 6. 总结

在这里，我们了解了Connect 4游戏的玩法以及如何在Java中实现规则。