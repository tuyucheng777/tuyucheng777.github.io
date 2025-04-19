---
layout: post
title:  用Java实现二叉树
category: algorithms
copyright: algorithms
excerpt: 二叉树
---

## 1. 简介

在本教程中，我们将介绍Java中二叉树的实现。

为了本教程的目的，**我们将使用包含int值的[排序二叉树](https://www.baeldung.com/cs/binary-search-trees)**。

## 2. 二叉树

**二叉树是一种递归数据结构，其中每个节点最多可以有2个子节点**。

二叉树的一种常见类型是[二叉搜索树](https://www.baeldung.com/cs/bst-validation)，其中每个节点的值都大于或等于左子树中的节点值，并且小于或等于右子树中的节点值。

以下是这种二叉树的直观表示：

![](/assets/images/2025/algorithms/javabinarytree01.png)

对于实现，我们将使用一个辅助Node类来存储int值，并保留对每个子节点的引用：

```java
class Node {
    int value;
    Node left;
    Node right;

    Node(int value) {
        this.value = value;
        right = null;
        left = null;
    }
}
```

然后我们添加树的起始节点，通常称为根：

```java
public class BinaryTree {

    Node root;

    // ...
}
```

## 3. 常用操作

现在让我们看看可以在二叉树上执行的最常见的操作。

### 3.1 插入元素

我们要介绍的第一个操作是插入新节点。

首先，**为了保持树的有序性，我们必须找到添加新节点的位置**。从根节点开始，我们将遵循以下规则：

- 如果新节点的值小于当前节点的值，则转到左子节点
- 如果新节点的值大于当前节点的值，则转到右子节点
- 当当前节点为空时，我们到达了叶节点，我们可以在该位置插入新节点

然后我们将创建一个递归方法来进行插入：

```java
private Node addRecursive(Node current, int value) {
    if (current == null) {
        return new Node(value);
    }

    if (value < current.value) {
        current.left = addRecursive(current.left, value);
    } else if (value > current.value) {
        current.right = addRecursive(current.right, value);
    } else {
        // value already exists
        return current;
    }

    return current;
}
```

接下来我们将创建从根节点开始递归的公共方法：

```java
public void add(int value) {
    root = addRecursive(root, value);
}
```

让我们看看如何使用这种方法从我们的示例中创建树：

```java
private BinaryTree createBinaryTree() {
    BinaryTree bt = new BinaryTree();

    bt.add(6);
    bt.add(4);
    bt.add(8);
    bt.add(3);
    bt.add(5);
    bt.add(7);
    bt.add(9);

    return bt;
}
```

### 3.2 查找元素

现在让我们添加一个方法来检查树是否包含特定的值。

和以前一样，我们首先创建一个遍历树的递归方法：

```java
private boolean containsNodeRecursive(Node current, int value) {
    if (current == null) {
        return false;
    }
    if (value == current.value) {
        return true;
    }
    return value < current.value
            ? containsNodeRecursive(current.left, value)
            : containsNodeRecursive(current.right, value);
}
```

在这里，我们通过将值与当前节点中的值进行比较来搜索值；然后，我们将根据结果继续在左子节点或右子节点中执行操作。

接下来我们将创建从根开始的公共方法：

```java
public boolean containsNode(int value) {
    return containsNodeRecursive(root, value);
}
```

然后我们将创建一个简单的测试来验证树是否确实包含插入的元素：

```java
@Test
public void givenABinaryTree_WhenAddingElements_ThenTreeContainsThoseElements() {
    BinaryTree bt = createBinaryTree();

    assertTrue(bt.containsNode(6));
    assertTrue(bt.containsNode(4));
 
    assertFalse(bt.containsNode(1));
}
```

所有添加的节点都应包含在树中。

### 3.3 删除元素

另一个常见操作是从树中删除节点。

首先，我们必须以与之前类似的方式找到要删除的节点：

```java
private Node deleteRecursive(Node current, int value) {
    if (current == null) {
        return null;
    }

    if (value == current.value) {
        // Node to delete found
        // ... code to delete the node will go here
    }
    if (value < current.value) {
        current.left = deleteRecursive(current.left, value);
        return current;
    }
    current.right = deleteRecursive(current.right, value);
    return current;
}
```

一旦我们找到要删除的节点，主要有3种不同的情况：

- **节点没有子节点**：这是最简单的情况；我们只需要在其父节点中用null替换该节点
- **节点只有一个子节点**：在父节点中，我们用其唯一的子节点替换该节点。
- **节点有两个子节点**：这是最复杂的情况，因为它需要树重组

让我们看看当节点是叶节点时的情况：

```java
if (current.left == null && current.right == null) {
    return null;
}
```

现在让我们继续讨论节点有一个子节点的情况：

```java
if (current.right == null) {
    return current.left;
}

if (current.left == null) {
    return current.right;
}
```

这里我们返回非空子节点，以便可以将其分配给父节点。

最后，我们必须处理节点有两个子节点的情况。

首先，我们需要找到将要替换被删除节点的节点，我们将使用即将被删除节点右子树的最小节点：

```java
private int findSmallestValue(Node root) {
    return root.left == null ? root.value : findSmallestValue(root.left);
}
```

然后我们将最小的值分配给要删除的节点，之后我们将它从右子树中删除：

```java
int smallestValue = findSmallestValue(current.right);
current.value = smallestValue;
current.right = deleteRecursive(current.right, smallestValue);
return current;
```

最后，我们将创建从根开始删除的公共方法：

```java
public void delete(int value) {
    root = deleteRecursive(root, value);
}
```

现在让我们检查删除是否按预期进行：

```java
@Test
public void givenABinaryTree_WhenDeletingElements_ThenTreeDoesNotContainThoseElements() {
    BinaryTree bt = createBinaryTree();

    assertTrue(bt.containsNode(9));
    bt.delete(9);
    assertFalse(bt.containsNode(9));
}
```

## 4. 遍历树

在本节中，我们将探讨遍历树的不同方法，详细介绍深度优先和广度优先搜索。

我们将使用之前使用的相同树，并检查每种情况的遍历顺序。

### 4.1 深度优先搜索

**深度优先搜索是一种遍历类型，在探索下一个兄弟节点之前，它会尽可能深入地遍历每个子节点**。

执行深度优先搜索有几种方法：中序、前序和后序。

**中序遍历首先访问左子树，然后访问根节点，最后访问右子树**：

```java
public void traverseInOrder(Node node) {
    if (node != null) {
        traverseInOrder(node.left);
        System.out.print(" " + node.value);
        traverseInOrder(node.right);
    }
}
```

如果我们调用此方法，控制台输出将显示有序遍历：

```text
3 4 5 6 7 8 9
```

**前序遍历首先访问根节点，然后访问左子树，最后访问右子树**：

```java
public void traversePreOrder(Node node) {
    if (node != null) {
        System.out.print(" " + node.value);
        traversePreOrder(node.left);
        traversePreOrder(node.right);
    }
}
```

让我们在控制台输出中检查一下前序遍历：

```text
6 4 3 5 8 7 9
```

**后序遍历访问左子树、右子树，最后访问根节点**：

```java
public void traversePostOrder(Node node) {
    if (node != null) {
        traversePostOrder(node.left);
        traversePostOrder(node.right);
        System.out.print(" " + node.value);
    }
}
```

以下是按后序排列的节点：

```text
3 5 4 7 9 8 6
```

### 4.2 广度优先搜索

这是另一种常见的遍历类型，**即在进入下一级之前访问某一级别的所有节点**。

这种遍历也称为级别顺序，从根开始从左到右访问树的所有级别。

在实现上，我们将使用一个队列来按顺序保存每一层的节点。我们将从列表中提取每个节点，打印其值，然后将其子节点添加到队列中：

```java
public void traverseLevelOrder() {
    if (root == null) {
        return;
    }

    Queue<Node> nodes = new LinkedList<>();
    nodes.add(root);

    while (!nodes.isEmpty()) {
        Node node = nodes.remove();

        System.out.print(" " + node.value);

        if (node.left != null) {
            nodes.add(node.left);
        }

        if (node.right != null) {
            nodes.add(node.right);
        }
    }
}
```

在这种情况下，节点的顺序将是：

```text
6 4 8 3 5 7 9
```

## 5. 总结

在本文中，我们学习了如何在Java中实现排序二叉树及其最常见的操作。