---
layout: post
title:  在Java中反转二叉树
category: algorithms
copyright: algorithms
excerpt: 树
---

## 1. 概述

**反转二叉树是我们在技术面试中可能会被要求解决的问题之一**。

在此快速教程中，我们将看到解决此问题的几种不同方法。

## 2. 二叉树

**二叉树是一种数据结构，其中每个元素最多有两个子节点**，称为左子节点和右子节点。树的顶部元素是根节点，**而子节点是内部节点**。

然而，如果一个节点没有子节点，它就被称为叶子节点。

话虽如此，让我们创建代表节点的对象：
```java
public class TreeNode {

    private int value;
    private TreeNode rightChild;
    private TreeNode leftChild;

    // Getters and setters
}
```

然后，让我们创建我们将在示例中使用的树：
```java
TreeNode leaf1 = new TreeNode(1);
TreeNode leaf2 = new TreeNode(3);
TreeNode leaf3 = new TreeNode(6);
TreeNode leaf4 = new TreeNode(9);

TreeNode nodeRight = new TreeNode(7, leaf3, leaf4);
TreeNode nodeLeft = new TreeNode(2, leaf1, leaf2);

TreeNode root = new TreeNode(4, nodeLeft, nodeRight);
```

在之前的方法中，我们创建了以下结构：

![](/assets/images/2025/algorithms/javareversingabinarytree01.png)

通过从左到右反转树，我们最终会得到以下结构：

![](/assets/images/2025/algorithms/javareversingabinarytree02.png)

## 3. 反转二叉树

### 3.1 递归方法

**在第一个例子中，我们将使用递归来反转树**。

首先，**我们将使用树的根调用我们的方法，然后分别将其应用于左子节点和右子节点**，直到到达树的叶子：
```java
public void reverseRecursive(TreeNode treeNode) {
    if(treeNode == null) {
        return;
    }

    TreeNode temp = treeNode.getLeftChild();
    treeNode.setLeftChild(treeNode.getRightChild());
    treeNode.setRightChild(temp);

    reverseRecursive(treeNode.getLeftChild());
    reverseRecursive(treeNode.getRightChild());
}
```

### 3.2 迭代法

在第二个示例中，**我们将使用迭代方法反转树**。为此，**我们将使用LinkedList，并使用树的根初始化它**。

然后，**对于我们从列表轮询的每个节点，我们在对它们进行排列之前，将其子节点添加到该列表中**。

我们不断地在LinkedList中添加和删除，直到到达树的叶子：
```java
public void reverseIterative(TreeNode treeNode) {
    List<TreeNode> queue = new LinkedList<>();

    if(treeNode != null) {
        queue.add(treeNode);
    }

    while(!queue.isEmpty()) {
        TreeNode node = queue.poll();
        if(node.getLeftChild() != null){
            queue.add(node.getLeftChild());
        }
        if(node.getRightChild() != null){
            queue.add(node.getRightChild());
        }

        TreeNode temp = node.getLeftChild();
        node.setLeftChild(node.getRightChild());
        node.setRightChild(temp);
    }
}
```

## 4. 总结

在这篇简短的文章中，我们探讨了反转二叉树的两种方法。我们首先使用递归方法来反转它；然后，我们最终使用迭代方法来实现相同的效果。