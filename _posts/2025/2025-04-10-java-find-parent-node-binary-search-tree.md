---
layout: post
title:  使用Java在二叉搜索树中查找节点的父节点
category: algorithms
copyright: algorithms
excerpt: 二叉搜索树
---

## 1. 简介

[二叉搜索树](https://www.baeldung.com/cs/binary-search-trees)(BST)是一种帮助我们有效解决[实际问题](https://www.baeldung.com/cs/applications-of-binary-trees)的数据结构。

在这篇文章中，我们将研究如何解决在BST中查找节点父节点的问题。

## 2. 什么是二叉搜索树？

**BST是一棵树，其中每个节点最多指向两个节点，通常称为左子节点和右子节点。此外，每个节点的值都大于左子节点，小于右子节点**。

例如，我们假设有3个节点，A=2、B=1和C=4。因此，一个可能的BST以A为根，B为其左子节点，C为其右子节点。

在接下来的部分中，我们将使用通过默认insert()方法[实现的BST结构](https://www.baeldung.com/java-binary-tree)来解决查找节点父节点的问题。

## 3. 二叉搜索树中节点的父节点

在接下来的章节中，我们将描述在BST中查找节点的父节点的问题，并运用一些方法来解决它。

### 3.1 问题描述

正如我们在整篇文章中看到的，BST的给定节点具有指向其左子节点和右子节点的指针。

例如，让我们想象一个具有3个节点的简单BST：

![](/assets/images/2025/algorithms/javafindparentnodebinarysearchtree01.png)

节点8包含两个子节点，分别为5和12。因此，节点8是节点5和12的父节点。

**问题在于找到任何给定节点值的父节点**。换句话说，我们必须找到其任意子节点等于目标值的节点。例如，在上图的BST中，如果我们在程序中输入5，我们期望输出8。如果我们输入12，我们也期望输出8。

这个问题的极端情况是找到最顶层根节点的父节点，或者找到二叉搜索树中不存在的节点的父节点，这两种情况都没有父节点。

### 3.2 测试结构

在深入研究各种解决方案之前，让我们首先为测试定义一个基本结构：
```java
class BinaryTreeParentNodeFinderUnitTest {

    TreeNode subject;

    @BeforeEach
    void setUp() {
        subject = new TreeNode(8);
        subject.insert(5);
        subject.insert(12);
        subject.insert(3);
        subject.insert(7);
        subject.insert(1);
        subject.insert(4);
        subject.insert(11);
        subject.insert(14);
        subject.insert(13);
        subject.insert(16);
    }
}
```

BinaryTreeParentNodeFinderUnitTest定义了一个setUp()方法，用于创建以下BST：

![](/assets/images/2025/algorithms/javafindparentnodebinarysearchtree02.png)

## 4. 实现递归解决方案

**这个问题的直接解决方案是使用递归遍历树并提前返回其任何子节点等于目标值的节点**。

我们首先在TreeNode类中定义一个公共方法：
```java
TreeNode parent(int target) throws NoSuchElementException {
    return parent(this, new TreeNode(target));
}
```

现在，让我们定义TreeNode类中parent()方法的递归版本：
```java
TreeNode parent(TreeNode current, TreeNode target) throws NoSuchElementException {
    if (target.equals(current) || current == null) {
        throw new NoSuchElementException(format("No parent node found for 'target.value=%s' " +
                        "The target is not in the tree or the target is the topmost root node.",
                target.value));
    }

    if (target.equals(current.left) || target.equals(current.right)) {
        return current;
    }

    return parent(target.value < current.value ? current.left : current.right, target);
}
```

该算法首先检查当前节点是否是最顶层的根节点，或者该节点是否在树中不存在。在这两种情况下，该节点都没有父节点，因此我们抛出NoSuchElementException。

然后，算法检查当前节点是否有任何子节点等于目标节点。如果是，则当前节点是目标节点的父节点。因此，我们返回当前节点。

最后，我们根据目标值，使用向左或向右的递归调用来遍历BST。

让我们测试一下递归解决方案：
```java
@Test
void givenBinaryTree_whenFindParentNode_thenReturnCorrectParentNode() {
    assertThrows(NoSuchElementException.class, () -> subject.parent(1231));
    assertThrows(NoSuchElementException.class, () -> subject.parent(8));
    assertEquals(8, subject.parent(5).value);
    assertEquals(5, subject.parent(3).value);
    assertEquals(5, subject.parent(7).value);
    assertEquals(3, subject.parent(4).value);
    // assertions for other nodes
}
```

在最坏的情况下，该算法最多执行n次递归操作来查找父节点，每次操作的复杂度为O(1)，其中n是BST中的节点数；因此，它的[时间复杂度](https://www.baeldung.com/cs/time-vs-space-complexity#what-is-time-complexity)为O(n)。在[均衡的BST](https://www.baeldung.com/cs/self-balancing-bts)中，由于其[高度始终最多为log n](https://www.baeldung.com/cs/height-balanced-tree)，因此时间复杂度降至O(log n)。

此外，该算法使用[堆空间](https://www.baeldung.com/java-stack-heap#heap-space-in-java)进行递归调用。因此，在最坏的情况下，递归调用会在找到叶节点时停止。因此，该算法最多堆叠h次递归调用，这使得[空间复杂度](https://www.baeldung.com/cs/time-vs-space-complexity#what-is-space-complexity)为O(h)，其中h是BST的高度。

## 5. 实现迭代解决方案

几乎所有的[递归解决方案都有一个迭代版本](https://www.baeldung.com/cs/convert-recursion-to-iteration)，**具体来说，我们还可以使用栈和while循环(而不是递归)来查找BST的父级**。

为此，让我们将iterativeParent()方法添加到TreeNode类：
```java
TreeNode iterativeParent(int target) {
    return iterativeParent(this, new TreeNode(target));
}
```

上面的方法只是下面辅助方法的接口：
```java
TreeNode iterativeParent(TreeNode current, TreeNode target) {
    Deque <TreeNode> parentCandidates = new LinkedList<>();

    String notFoundMessage = format("No parent node found for 'target.value=%s' " +
                    "The target is not in the tree or the target is the topmost root node.",
            target.value);

    if (target.equals(current)) {
        throw new NoSuchElementException(notFoundMessage);
    }

    while (current != null || !parentCandidates.isEmpty()) {
        while (current != null) {
            parentCandidates.addFirst(current);
            current = current.left;
        }

        current = parentCandidates.pollFirst();

        if (target.equals(current.left) || target.equals(current.right)) {
            return current;
        }

        current = current.right;
    }

    throw new NoSuchElementException(notFoundMessage);
}
```

该算法首先初始化一个栈来存储父候选，然后它主要取决于4个主要部分：

1. 外层while循环检查我们是否正在访问非叶节点，或者父节点候选栈是否不为空。在这两种情况下，我们都应该继续遍历BST，直到找到目标父节点。
2. 内层while循环再次检查我们是否正在访问非叶节点。此时，由于我们使用的是中序遍历，因此访问非叶子节点意味着我们应该先左遍历。因此，我们将父候选节点添加到栈中，然后继续左遍历。
3. 访问完左节点后，我们从Deque中轮询一个节点，检查该节点是否是目标节点的父节点，如果是，则返回该节点。如果找不到父节点，则继续向右遍历。
4. 最后，如果主循环完成而没有返回任何节点，我们可以假设该节点不存在或者它是最顶层的根节点。

现在，让我们测试迭代方法：
```java
@Test
void givenBinaryTree_whenFindParentNodeIteratively_thenReturnCorrectParentNode() {
    assertThrows(NoSuchElementException.class, () -> subject.iterativeParent(1231));
    assertThrows(NoSuchElementException.class, () -> subject.iterativeParent(8));
    assertEquals(8, subject.iterativeParent(5).value);
    assertEquals(5, subject.iterativeParent(3).value);
    assertEquals(5, subject.iterativeParent(7).value);
    assertEquals(3, subject.iterativeParent(4).value);

    // assertion for other nodes
}
```

在最坏的情况下，我们需要遍历整个o树来找到父节点，这使得迭代解决方案的空间复杂度为O(n)。同样，如果BST是平衡的，我们可以在O(log n)的复杂度内完成同样的操作。

当到达叶节点时，我们开始从parentCandidates栈中轮询元素。因此，用于存储父候选的附加栈最多包含h个元素，其中h是BST的高度。因此，它的空间复杂度也是O(h)。

## 6. 使用父指针创建BST

该问题的另一种解决方案是修改现有的BST数据结构来存储每个节点的父节点。

为此，让我们创建另一个名为ParentKeeperTreeNode的类，并添加一个名为parent的新字段：
```java
class ParentKeeperTreeNode {

    int value;
    ParentKeeperTreeNode parent;
    ParentKeeperTreeNode left;
    ParentKeeperTreeNode right;

    // value field arg constructor

    // equals and hashcode
}
```

现在，我们需要创建一个自定义insert()方法来保存父节点：
```java
void insert(ParentKeeperTreeNode currentNode, final int value) {
    if (currentNode.left == null && value < currentNode.value) {
        currentNode.left = new ParentKeeperTreeNode(value);
        currentNode.left.parent = currentNode;
        return;
    }

    if (currentNode.right == null && value > currentNode.value) {
        currentNode.right = new ParentKeeperTreeNode(value);
        currentNode.right.parent = currentNode;
        return;
    }

    if (value > currentNode.value) {
        insert(currentNode.right, value);
    }

    if (value < currentNode.value) {
        insert(currentNode.left, value);
    }
}
```

**insert()方法在为当前节点创建新的左子节点或右子节点时，也会保存父节点。在这种情况下，由于我们创建的是新子节点，因此父节点始终是我们正在访问的当前节点**。

最后，我们可以测试存储父指针的BST版本：
```java
@Test
void givenParentKeeperBinaryTree_whenGetParent_thenReturnCorrectParent() {
    ParentKeeperTreeNode subject = new ParentKeeperTreeNode(8);
    subject.insert(5);
    subject.insert(12);
    subject.insert(3);
    subject.insert(7);
    subject.insert(1);
    subject.insert(4);
    subject.insert(11);
    subject.insert(14);
    subject.insert(13);
    subject.insert(16);

    assertNull(subject.parent);
    assertEquals(8, subject.left.parent.value);
    assertEquals(8, subject.right.parent.value);
    assertEquals(5, subject.left.left.parent.value);
    assertEquals(5, subject.left.right.parent.value);

    // tests for other nodes
}
```

在这种类型的BST中，我们在插入节点时计算父节点。因此，为了验证结果，我们只需检查每个节点的父节点引用即可。

因此，我们无需在O(h)的时间复杂度内计算每个给定节点的parent()，而是可以在O(1)时间内通过引用立即获取它。此外，每个节点的父节点只是对内存中另一个现有对象的引用，因此，空间复杂度也是O(1)。

**当我们经常需要检索节点的父节点时，该版本的BST很有用，因为parent()操作已经过优化**。

## 7. 总结

在这篇文章中，我们看到了寻找BST中任何给定节点的父节点的问题。

我们通过代码示例练习了3种解决方案，一种使用递归遍历BST，另一种使用栈存储父候选节点并遍历BST，最后一种方法在每个节点中保留父引用，以便在常数时间内获取父节点。