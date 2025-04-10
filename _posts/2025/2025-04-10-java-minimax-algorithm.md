---
layout: post
title:  Minimax算法简介及其Java实现
category: algorithms
copyright: algorithms
excerpt: Minimax算法
---

## 1. 概述

在本文中，我们将讨论[Minimax算法](https://www.baeldung.com/cs/minimax-algorithm)及其在AI中的应用，由于它是一种博弈论算法，我们将使用它来实现一个简单的游戏。

我们还将讨论使用该算法的优点并了解如何改进它。

## 2. 简介

Minimax是一种决策算法，**通常用于回合制双人游戏**，该算法的目标是找到最优的下一步行动。

在算法中，一个玩家被称为最大化者，另一个玩家被称为最小化者。如果我们为游戏板分配一个评估分数，一个玩家会尝试选择具有最高分数的游戏状态，而另一个玩家会选择具有最低分数的状态。

换句话说，**最大化器努力获得最高分数，而最小化器则尝试通过对抗动作来获得最低分数**。

它基于[零和博弈](https://en.wikipedia.org/wiki/Zero-sum_game)概念，**在零和博弈中，总效用得分由所有玩家平分，一个玩家得分的增加会导致另一个玩家得分的减少**。因此，总得分始终为0。一个玩家获胜，另一个玩家必须输。这类游戏的例子包括国际象棋、扑克、西洋跳棋和井字棋。

一个有趣的事实-1997年，IBM的国际象棋计算机“深蓝”(使用Minimax构建)击败了加里卡斯帕罗夫(国际象棋世界冠军)。

## 3. Minimax算法

我们的目标是找到最适合玩家的走法，为此，我们可以选择评估分数最高的节点。为了让这个过程更加智能，我们还可以预测并评估潜在对手的走法。

对于每一步，我们可以在计算能力允许的范围内预测尽可能多的棋步，该算法假设对手的棋步处于最优状态。

从技术上讲，我们从根节点开始，选择最佳节点，我们根据节点的评估分数对其进行评估。在本例中，评估函数只能为结果节点(叶子)分配分数，因此，我们递归地到达具有分数的叶子并反向传播分数。

考虑下面的游戏树：

![](/assets/images/2025/algorithms/javaminimaxalgorithm01.png)

**最大化器从根节点开始**，选择得分最高的落子。遗憾的是，只有叶子节点有评估分数，因此算法必须递归地到达叶子节点。在给定的游戏树中，目前轮到最小化器从叶子节点中选择落子，因此得分最低的节点(此处为节点3和4)将被选中，它继续以类似的方式挑选最佳节点，直到到达根节点。

现在，让我们正式定义算法的步骤：

1. 构建完整的游戏树
2. 使用评估函数评估叶子的得分
3. 考虑到玩家类型，从叶子到根的备用分数：
   - 对于最大玩家，选择得分最高的孩子
   - 对于最小玩家，选择得分最小的孩子
4. 在根节点处，选择具有最大值的节点并执行相应的移动

## 4. 实现

现在，让我们实现一个游戏。

在游戏中，我们有一堆骨头，数量为n，两名玩家轮流拾取1、2或3根骨头，无法拾取任何骨头的玩家将输掉游戏，每个玩家都以最优方式进行游戏。给定n的值，让我们编写一个AI。

为了定义游戏规则，我们将实现GameOfBones类：
```java
class GameOfBones {
   static List<Integer> getPossibleStates(int noOfBonesInHeap) {
      return IntStream.rangeClosed(1, 3).boxed()
              .map(i -> noOfBonesInHeap - i)
              .filter(newHeapCount -> newHeapCount >= 0)
              .collect(Collectors.toList());
   }
}
```

此外，我们还需要Node和Tree类的实现：
```java
public class Node {
   int noOfBones;
   boolean isMaxPlayer;
   int score;
   List<Node> children;
   // setters and getters
}

public class Tree {
   Node root;
   // setters and getters
}
```

现在我们来实现这个算法，它需要一棵游戏树来预测并找到最佳走法，让我们来实现它：
```java
public class MiniMax {
   Tree tree;

   public void constructTree(int noOfBones) {
      tree = new Tree();
      Node root = new Node(noOfBones, true);
      tree.setRoot(root);
      constructTree(root);
   }

   private void constructTree(Node parentNode) {
      List<Integer> listofPossibleHeaps = GameOfBones.getPossibleStates(parentNode.getNoOfBones());
      boolean isChildMaxPlayer = !parentNode.isMaxPlayer();
      listofPossibleHeaps.forEach(n -> {
         Node newNode = new Node(n, isChildMaxPlayer);
         parentNode.addChild(newNode);
         if (newNode.getNoOfBones() > 0) {
            constructTree(newNode);
         }
      });
   }
}
```

现在，我们将实现checkWin方法，该方法将模拟一局游戏，为两个玩家选择最优走法。它将分数设置为：

- 如果最大化者获胜，则+1
- 如果最小化器获胜-1

如果第一个玩家(在我们的例子中是最大化者)获胜，checkWin将返回true：
```java
public boolean checkWin() {
   Node root = tree.getRoot();
   checkWin(root);
   return root.getScore() == 1;
}

private void checkWin(Node node) {
   List<Node> children = node.getChildren();
   boolean isMaxPlayer = node.isMaxPlayer();
   children.forEach(child -> {
      if (child.getNoOfBones() == 0) {
         child.setScore(isMaxPlayer ? 1 : -1);
      } else {
         checkWin(child);
      }
   });
   Node bestChild = findBestChild(isMaxPlayer, children);
   node.setScore(bestChild.getScore());
}
```

这里，如果玩家是最大化者，则findBestChild方法会找到得分最高的节点；否则，它返回得分最低的子节点：
```java
private Node findBestChild(boolean isMaxPlayer, List<Node> children) {
   Comparator<Node> byScoreComparator = Comparator.comparing(Node::getScore);
   return children.stream()
           .max(isMaxPlayer ? byScoreComparator : byScoreComparator.reversed())
           .orElseThrow(NoSuchElementException::new);
}
```

最后，让我们用一些n值(堆中的骨头数量)来实现一个测试用例：
```java
@Test
public void givenMiniMax_whenCheckWin_thenComputeOptimal() {
   miniMax.constructTree(6);
   boolean result = miniMax.checkWin();

   assertTrue(result);

   miniMax.constructTree(8);
   result = miniMax.checkWin();

   assertFalse(result);
}
```

## 5. 改进

对于大多数问题来说，构建一棵完整的博弈树是不可行的。**在实践中，我们可以开发一颗部分树(只构建到预定层数的树)**。

然后，我们必须实现一个评估函数，它应该能够决定玩家当前的状态有多好。

即使我们不构建完整的游戏树，计算具有高分支因子的游戏的动作也可能非常耗时。

**幸运的是，有一种方法可以找到最优解，而无需探索游戏树的每个节点**。我们可以遵循一些规则跳过一些分支，而不会影响最终结果，**这个过程称为剪枝**。[Alpha–beta剪枝](https://en.wikipedia.org/wiki/Alpha–beta_pruning)是Minimax算法的一种流行变体。

## 6. 总结

Minimax算法是计算机棋盘游戏中最流行的算法之一，它广泛应用于回合制游戏，当玩家掌握完整的游戏信息时，它可能是一个不错的选择。

对于分支因子极高的游戏(例如围棋)，它可能不是最佳选择。不过，如果实现得当，它可以成为一个相当智能的AI。