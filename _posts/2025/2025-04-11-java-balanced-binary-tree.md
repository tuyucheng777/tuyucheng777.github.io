---
layout: post
title:  如何在Java中判断二叉树是否平衡
category: algorithms
copyright: algorithms
excerpt: 二叉树
---

## 1. 概述

树是计算机科学中最重要的数据结构之一，我们通常对平衡树感兴趣，因为它具有宝贵的属性，其结构允许在对数时间内执行查询、插入、删除等操作。

在本教程中，我们将学习如何确定二叉树是否平衡。

## 2. 定义

首先，让我们介绍一些定义，以确保我们理解一致：

- [二叉树](https://www.baeldung.com/java-binary-tree)：一种树，每个节点有0个、1个或2个子节点
- 树的高度：从根到叶子的最大距离(与最深叶子的深度相同)
- 平衡树：一种树，**其中每个子树从根到任何叶子的最大距离最多比从根到任何叶子的最小距离大1**

下面是一个平衡二叉树的例子，三条绿色边简单直观地展示了如何[确定高度](https://www.baeldung.com/cs/height-balanced-tree)，而数字则表示层级。

![](/assets/images/2025/algorithms/javabalancedbinarytree01.png)

## 3. 域对象

那么，让我们从树的类开始：
```java
public class Tree {
    private int value;
    private Tree left;
    private Tree right;

    public Tree(int value, Tree left, Tree right) {
        this.value = value;
        this.left = left;
        this.right = right;
    }
}
```

为了简单起见，假设每个节点都有一个整数值。注意，**如果左树和右树都为空，则表示我们的节点是叶子节点**。

在介绍主要方法之前，让我们看看它应该返回什么：
```java
private class Result {
    private boolean isBalanced;
    private int height;

    private Result(boolean isBalanced, int height) {
        this.isBalanced = isBalanced;
        this.height = height;
    }
}
```

因此，对于每次调用，我们都会获得有关高度和平衡的信息。

## 4. 算法

有了平衡树的定义，我们就可以设计一个算法，**我们需要做的就是检查每个节点是否满足所需的属性**，这可以通过递归深度优先搜索遍历轻松实现。

现在，我们的递归方法将针对每个节点调用。此外，它将跟踪当前深度，每次调用都会返回有关高度和平衡的信息。

现在，让我们看一下深度优先方法：
```java
private Result isBalancedRecursive(Tree tree, int depth) {
    if (tree == null) {
        return new Result(true, -1);
    }

    Result leftSubtreeResult = isBalancedRecursive(tree.left(), depth + 1);
    Result rightSubtreeResult = isBalancedRecursive(tree.right(), depth + 1);

    boolean isBalanced = Math.abs(leftSubtreeResult.height - rightSubtreeResult.height) <= 1;
    boolean subtreesAreBalanced = leftSubtreeResult.isBalanced && rightSubtreeResult.isBalanced;
    int height = Math.max(leftSubtreeResult.height, rightSubtreeResult.height) + 1;

    return new Result(isBalanced && subtreesAreBalanced, height);
}
```

首先，我们需要考虑节点为空的情况：我们将返回true(这意味着树是平衡的)并且返回-1作为高度。

然后，**我们对左子树和右子树进行两次递归调用，保持深度保持最新**。

至此，我们已经完成了当前节点子节点的计算；现在，我们拥有检查平衡所需的所有数据：

- isBalanced变量检查子树的高度，并且
- substreesAreBalanced指示子树是否都平衡

最后，我们可以返回有关平衡和高度的信息，使用门面方法简化第一个递归调用可能也是一个好主意：
```java
public boolean isBalanced(Tree tree) {
    return isBalancedRecursive(tree, -1).isBalanced;
}
```

## 5. 总结

在本文中，我们讨论了如何确定二叉树是否平衡，我们解释了深度优先搜索方法。