---
layout: post
title:  Java中的深度优先搜索
category: algorithms
copyright: algorithms
excerpt: 搜索
---

## 1. 概述

在本教程中，我们将探索Java中的深度优先搜索。

[深度优先搜索(DFS)](https://www.baeldung.com/cs/depth-first-traversal-methods)是一种用于树和[图数据结构](https://www.baeldung.com/cs/graphs)的遍历算法。**深度优先搜索会深入每个分支，然后再探索另一个分支**。

在接下来的部分中，我们将首先了解树的实现，然后了解图的实现。

要了解如何在Java中实现这些结构，请查看我们之前关于[二叉树](https://www.baeldung.com/java-binary-tree)和[图](https://www.baeldung.com/java-graphs)的教程。

## 2. 树深度优先搜索

使用DFS遍历树有3种不同的顺序：

1. 前序遍历
2. 中序遍历
3. 后序遍历

### 2.1 前序遍历

**在前序遍历中，我们首先遍历根，然后遍历左子树和右子树**。

我们可以使用递归简单地实现前序遍历：

- 访问当前节点
- 遍历左子树
- 遍历右子树

```java
public void traversePreOrder(Node node) {
    if (node != null) {
        visit(node.value);
        traversePreOrder(node.left);
        traversePreOrder(node.right);
    }
}
```

我们还可以实现不使用递归的前序遍历。

为了实现迭代前序遍历，我们需要一个栈，遵循以下步骤：

- 将根推入栈

- 当栈不为空时

  - 弹出当前节点
  - 访问当前节点
  - 将右子节点推入栈，然后将左子节点推入栈
  
```java
public void traversePreOrderWithoutRecursion() {
    Stack<Node> stack = new Stack<Node>();
    Node current = root;
    stack.push(root);
    while(!stack.isEmpty()) {
        current = stack.pop();
        visit(current.value);
        
        if(current.right != null) {
            stack.push(current.right);
        }    
        if(current.left != null) {
            stack.push(current.left);
        }
    }        
}
```

### 2.2 中序遍历

对于中序遍历，**我们首先遍历左子树，然后遍历根，最后遍历右子树**。

二叉搜索树的中序遍历意味着按照节点值的升序遍历节点。

我们可以用递归的方式简单地实现中序遍历：
```java
public void traverseInOrder(Node node) {
    if (node != null) {
        traverseInOrder(node.left);
        visit(node.value);
        traverseInOrder(node.right);
    }
}
```

我们也可以实现不使用递归的中序遍历：

- 用根初始化当前节点

- 当当前节点不为空或栈不为空时

  - 继续将左子节点推入栈，直到到达当前节点的最左子节点
  - 弹出并访问栈最左边的节点
  - 将当前节点设置为弹出节点的右子节点
  
```java
public void traverseInOrderWithoutRecursion() {
    Stack stack = new Stack<>();
    Node current = root;

    while (current != null || !stack.isEmpty()) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }

        Node top = stack.pop();
        visit(top.value);
        current = top.right;
    }
}
```

### 2.3 后序遍历

最后，在后序遍历中，**我们在遍历根之前先遍历左子树和右子树**。

我们可以遵循之前的递归解决方案：
```java
public void traversePostOrder(Node node) {
    if (node != null) {
        traversePostOrder(node.left);
        traversePostOrder(node.right);
        visit(node.value);
    }
}
```

或者，我们也可以实现不使用递归的后序遍历：

- 将根节点压入栈

- 当栈不为空时

  - 检查我们是否已经遍历了左子树和右子树
  - 如果不是则将右子节点和左子节点推入栈
  
```java
public void traversePostOrderWithoutRecursion() {
    Stack<Node> stack = new Stack<Node>();
    Node prev = root;
    Node current = root;
    stack.push(root);

    while (!stack.isEmpty()) {
        current = stack.peek();
        boolean hasChild = (current.left != null || current.right != null);
        boolean isPrevLastChild = (prev == current.right || (prev == current.left && current.right == null));

        if (!hasChild || isPrevLastChild) {
            current = stack.pop();
            visit(current.value);
            prev = current;
        } else {
            if (current.right != null) {
                stack.push(current.right);
            }
            if (current.left != null) {
                stack.push(current.left);
            }
        }
    }   
}
```

## 3. 图深度优先搜索

图和树之间的主要区别在于**图可能包含循环**。

因此，为了避免循环搜索，我们在访问每个节点时会对其进行标记。

我们将看到图DFS的两种实现：使用递归的和不使用递归。

### 3.1 使用递归的图DFS

首先，让我们从简单的递归开始：

- 我们将从给定节点开始
- 将当前节点标记为已访问
- 访问当前节点
- 遍历未访问的相邻顶点

```java
public boolean[] dfs(int start) {
    boolean[] isVisited = new boolean[adjVertices.size()];
    return dfsRecursive(start, isVisited);
}

private boolean[] dfsRecursive(int current, boolean[] isVisited) {
    isVisited[current] = true;
    visit(current);
    for (int dest : adjVertices.get(current)) {
        if (!isVisited[dest])
            dfsRecursive(dest, isVisited);
    }
    return isVisited;
}
```

### 3.2 无递归的图DFS

我们也可以不使用递归来实现图DFS，我们只需使用栈：

- 我们将从给定节点开始

- 将起始节点压入栈

- 当栈不为空时

  - 将当前节点标记为已访问
  - 访问当前节点
  - 推入未访问的相邻顶点

```java
public void dfsWithoutRecursion(int start) {
    Stack<Integer> stack = new Stack<Integer>();
    boolean[] isVisited = new boolean[adjVertices.size()];
    stack.push(start);
    while (!stack.isEmpty()) {
        int current = stack.pop();
        if(!isVisited[current]){
            isVisited[current] = true;
            visit(current);
            for (int dest : adjVertices.get(current)) {
                if (!isVisited[dest])
                    stack.push(dest);
            }
        }
    }
    return isVisited;
}
```

### 3.3 拓扑排序

图深度优先搜索有很多应用，其中一个著名的应用是拓扑排序。

**有向图的拓扑排序是对其顶点的线性排序，以便对于每个边，源节点都位于目标节点之前**。

为了进行拓扑排序，我们需要对刚刚实现的DFS进行一个简单的补充：

- 我们需要将访问过的顶点保存在栈中，因为拓扑排序是按相反顺序访问顶点
- 遍历完所有邻居后，我们才将访问的节点推送到栈


```java
public List<Integer> topologicalSort(int start) {
    LinkedList<Integer> result = new LinkedList<Integer>();
    boolean[] isVisited = new boolean[adjVertices.size()];
    topologicalSortRecursive(start, isVisited, result);
    return result;
}

private void topologicalSortRecursive(int current, boolean[] isVisited, LinkedList<Integer> result) {
    isVisited[current] = true;
    for (int dest : adjVertices.get(current)) {
        if (!isVisited[dest])
            topologicalSortRecursive(dest, isVisited, result);
    }
    result.addFirst(current);
}
```

## 4. 总结

在本文中，我们讨论了树和图数据结构的深度优先搜索。