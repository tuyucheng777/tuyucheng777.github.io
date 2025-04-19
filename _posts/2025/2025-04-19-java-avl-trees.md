---
layout: post
title:  Java中的AVL树指南
category: algorithms
copyright: algorithms
excerpt: 树
---

## 1. 简介

在本教程中，我们将介绍AVL树，并研究插入、删除和搜索值的算法。

## 2. 什么是AVL树？

AVL树以其发明者Adelson-Velsky和Landis的名字命名，是一种自平衡[二叉搜索树](https://www.baeldung.com/java-binary-tree)(BST)。

**自平衡树是一种二叉搜索树，它按照一定的平衡规则在插入和删除后平衡高度**。

二叉搜索树(BST)的最坏情况时间复杂度取决于树的高度，具体来说，就是从树的根节点到某个节点的最长路径。对于一个有N个节点的二叉搜索树(BST)，假设每个节点只有0个或1个子节点。因此，它的高度等于N，最坏情况下的搜索时间为O(N)。因此，我们设计二叉搜索树(BST)的主要目标是使最大高度接近log(N)。

节点N的平衡因子是height(right(N)) – height(left(N))，**在AVL树中，节点的平衡因子只能是1、0或-1中的一个值**。

让我们为树定义一个Node对象：

```java
public class Node {
    int key;
    int height;
    Node left;
    Node right;
    // ...
}
```

接下来，让我们定义AVLTree：

```java
public class AVLTree {

    private Node root;

    void updateHeight(Node n) {
        n.height = 1 + Math.max(height(n.left), height(n.right));
    }

    int height(Node n) {
        return n == null ? -1 : n.height;
    }

    int getBalance(Node n) {
        return (n == null) ? 0 : height(n.right) - height(n.left);
    }

    // ...
}
```

## 3. 如何平衡AVL树？

AVL树在插入或删除节点后会检查其节点的平衡因子，如果节点的平衡因子大于1或小于-1，则树会重新平衡自身。

重新平衡树有两种操作：

- 右旋
- 左旋

### 3.1 右旋

让我们从右旋开始。

假设我们有一个名为T1的BST，其中Y为根节点，X为Y的左孩子，Z为X的右孩子。给定BST的特性，我们知道X < Z < Y。

在对Y进行右旋转之后，我们得到一棵树，称为T2，其中X为根，Y为X的右孩子，Z为Y的左孩子。T2仍然是BST，因为它保持了X < Z < Y的顺序。

![](/assets/images/2025/algorithms/javaavltrees01.png)

让我们看一下AVLTree的右旋操作：

```java
Node rotateRight(Node y) {
    Node x = y.left;
    Node z = x.right;
    x.right = y;
    y.left = z;
    updateHeight(y);
    updateHeight(x);
    return x;
}
```

### 3.2 左旋

假设有一个名为T1的BST，其中Y为根节点，X为Y的右孩子，Z为X的左孩子。由此，我们知道Y < Z < X。

Y左旋转后，我们得到一棵树T2，其中X为根，Y为X的左孩子，Z为Y的右孩子。T2仍然是BST，因为它保持了Y < Z < X的顺序。

![](/assets/images/2025/algorithms/javaavltrees02.png)

让我们看一下AVLTree的左旋操作：

```java
Node rotateLeft(Node y) {
    Node x = y.right;
    Node z = x.left;
    x.left = y;
    y.right = z;
    updateHeight(y);
    updateHeight(x);
    return x;
}
```

### 3.3 重平衡技术

**我们可以将右旋和左旋操作以更复杂的组合使用，以使AVL树在其节点发生任何更改后仍保持平衡**。在不平衡的结构中，至少有一个节点的平衡因子等于2或-2，让我们看看如何在这些情况下平衡树。

当节点Z的平衡因子为2时，以Z为根的子树处于这两种状态之一，将Y视为Z的右孩子。

对于第一种情况，Y的右孩子(X)的高度大于左孩子(T2)的高度，我们可以通过Z的左旋轻松地重新平衡树。

![](/assets/images/2025/algorithms/javaavltrees03.png)

对于第二种情况，Y的右孩子(T4)的高度小于左孩子(X)的高度，这种情况需要组合使用旋转操作。

![](/assets/images/2025/algorithms/javaavltrees04.png)

在这种情况下，我们首先将Y轴向右旋转，使树的形状与上一种情况相同。然后，我们可以通过Z向左旋转来重新平衡树。

另外，当节点Z的平衡因子为-2时，其子树处于这两种状态之一，因此我们将Z视为根，将Y视为其左孩子。

Y的左孩子的高度大于其右孩子的高度，因此我们通过Z的右旋转来平衡树。

![](/assets/images/2025/algorithms/javaavltrees05.png)

或者在第二种情况下，Y的右孩子的高度大于其左孩子的高度。

![](/assets/images/2025/algorithms/javaavltrees06.png)

因此，首先，我们通过Y左旋转将其转换为以前的形状，然后通过Z右旋转使树保持平衡。

让我们看一下AVLTree的重新平衡操作：

```java
Node rebalance(Node z) {
    updateHeight(z);
    int balance = getBalance(z);
    if (balance > 1) {
        if (height(z.right.right) > height(z.right.left)) {
            z = rotateLeft(z);
        } else {
            z.right = rotateRight(z.right);
            z = rotateLeft(z);
        }
    } else if (balance < -1) {
        if (height(z.left.left) > height(z.left.right))
            z = rotateRight(z);
        else {
            z.left = rotateLeft(z.left);
            z = rotateRight(z);
        }
    }
    return z;
}
```

**在插入或删除一个节点后，我们将对从更改的节点到根的路径中的所有节点使用重平衡**。

## 4. 插入节点

当我们要在树中插入一个键时，我们必须找到它的正确位置以符合二叉搜索树(BST)规则。因此，我们从根节点开始，将其值与新键进行比较。如果键值更大，则继续向右移动；否则，则从左孩子节点移动。

一旦我们找到合适的父节点，我们就会根据值将新键作为节点添加到左侧或右侧。

插入节点后，我们得到了一棵BST，但它可能不是AVL树。因此，我们检查平衡因子，并针对从新节点到根路径上的所有节点重新平衡BST。

我们来看一下插入操作：

```java
Node insert(Node root, int key) {
    if (root == null) {
        return new Node(key);
    } else if (root.key > key) {
        root.left = insert(root.left, key);
    } else if (root.key < key) {
        root.right = insert(root.right, key);
    } else {
        throw new RuntimeException("duplicate Key!");
    }
    return rebalance(root);
}
```

**重要的是要记住，键在树中是唯一的-没有两个节点共享相同的键**。

插入算法的时间复杂度是高度的函数，由于我们的树是平衡的，我们可以假设最坏情况下的时间复杂度为O(log(N))。

## 5. 删除节点

要从树中删除一个键，我们首先必须在BST中找到它。

找到节点(称为Z)后，我们必须引入新的候选节点来替代它在树中的位置。如果Z是叶子节点，则候选节点为空。如果Z只有一个子节点，则该子节点就是候选节点；但如果Z有两个子节点，则过程会稍微复杂一些。

假设Z的右孩子名为Y，首先，我们找到Y的最左边的节点并将其称为X。然后，我们将Z的新值设置为等于X的值，并继续从Y中删除X。

最后，我们在最后调用重平衡方法来保持BST为AVL树。

这是我们的删除方法：

```java
Node delete(Node node, int key) {
    if (node == null) {
        return node;
    } else if (node.key > key) {
        node.left = delete(node.left, key);
    } else if (node.key < key) {
        node.right = delete(node.right, key);
    } else {
        if (node.left == null || node.right == null) {
            node = (node.left == null) ? node.right : node.left;
        } else {
            Node mostLeftChild = mostLeftChild(node.right);
            node.key = mostLeftChild.key;
            node.right = delete(node.right, node.key);
        }
    }
    if (node != null) {
        node = rebalance(node);
    }
    return node;
}
```

删除算法的时间复杂度是树的高度的函数，与插入方法类似，我们可以假设最坏情况下的时间复杂度为O(log(N))。

## 6. 搜索节点

在AVL树中搜索节点与任何BST中搜索节点相同。

从树的根节点开始，将键与节点的值进行比较。如果键等于值，则返回该节点。如果键大于值，则从右子节点开始搜索，否则继续从左子节点开始搜索。

搜索的时间复杂度是高度的函数，我们可以假设最坏情况下的时间复杂度为O(log(N))。

我们来看示例代码：

```java
Node find(int key) {
    Node current = root;
    while (current != null) {
        if (current.key == key) {
            break;
        }
        current = current.key < key ? current.right : current.left;
    }
    return current;
}
```

## 7. 总结

在本教程中，我们实现了具有插入、删除和搜索操作的AVL树。