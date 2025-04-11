---
layout: post
title:  在Java中查找链表的中间元素
category: algorithms
copyright: algorithms
excerpt: 链表
---

## 1. 概述

在本教程中，我们将解释如何在Java中查找链表的中间元素。

我们将在下一部分中介绍主要问题，并展示解决这些问题的不同方法。

## 2. 跟踪大小

这个问题很容易解决，只需在向列表添加新元素时跟踪其大小即可，如果我们知道大小，也就知道中间元素在哪里。

让我们看一个使用LinkedList的Java实现的示例：
```java
public static Optional<String> findMiddleElementLinkedList(LinkedList<String> linkedList) {
    if (linkedList == null || linkedList.isEmpty()) {
        return Optional.empty();
    }

    return Optional.of(linkedList.get((linkedList.size() - 1) / 2));
}
```

如果我们检查LinkedList类的内部代码，我们可以看到在这个例子中我们只是遍历列表直到到达中间元素：
```java
Node<E> node(int index) {
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++) {
            x = x.next;
        }
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--) {
            x = x.prev;
        }
        return x;
    }
}
```

## 3. 在不知道大小的情况下寻找中间值

我们经常会遇到这样的问题：我们**只有一个链表的头节点**，而我们需要找到中间元素。在这种情况下，我们不知道列表的大小，这使得这个问题更难解决。

我们将在下一节中展示解决此问题的几种方法，但首先，我们需要创建一个类来表示列表的一个节点。

让我们创建一个Node类，它存储字符串值：
```java
public static class Node {

    private Node next;
    private String data;

    // constructors/getters/setters

    public boolean hasNext() {
        return next != null;
    }

    public void setNext(Node next) {
        this.next = next;
    }

    public String toString() {
        return this.data;
    }
}
```

此外，我们将在测试用例中使用此辅助方法仅使用我们的节点创建单链表：
```java
private static Node createNodesList(int n) {
    Node head = new Node("1");
    Node current = head;

    for (int i = 2; i <= n; i++) {
        Node newNode = new Node(String.valueOf(i));
        current.setNext(newNode);
        current = newNode;
    }

    return head;
}
```

### 3.1 首先找到大小

解决这个问题最简单的方法是先找到列表的大小，然后按照我们之前使用的方法-迭代直到中间元素。

让我们看看这个解决方案的实际效果：
```java
public static Optional<String> findMiddleElementFromHead(Node head) {
    if (head == null) {
        return Optional.empty();
    }

    // calculate the size of the list
    Node current = head;
    int size = 1;
    while (current.hasNext()) {
        current = current.next();
        size++;
    }

    // iterate till the middle element
    current = head;
    for (int i = 0; i < (size - 1) / 2; i++) {
        current = current.next();
    }

    return Optional.of(current.data());
}
```

可以看出，**此代码对列表进行了两次迭代，因此，此解决方案性能较差，不推荐使用**。

### 3.2 迭代查找中间元素

我们现在要通过仅对列表进行一次迭代来找到中间元素，从而改进之前的解决方案。

为了迭代地实现这一点，我们需要两个指针同时遍历列表，**一个指针在每次迭代中前进2个节点，另一个指针在每次迭代中仅前进1个节点**。

当较快的指针到达列表的末尾时，较慢的指针将位于中间：
```java
public static Optional<String> findMiddleElementFromHead1PassIteratively(Node head) {
    if (head == null) {
        return Optional.empty();
    }

    Node slowPointer = head;
    Node fastPointer = head;

    while (fastPointer.hasNext() && fastPointer.next().hasNext()) {
        fastPointer = fastPointer.next().next();
        slowPointer = slowPointer.next();
    }

    return Optional.ofNullable(slowPointer.data());
}
```

我们可以使用包含奇数和偶数个元素的列表通过简单的单元测试来测试该解决方案：
```java
@Test
void whenFindingMiddleFromHead1PassIteratively_thenMiddleFound() {
    assertEquals("3", MiddleElementLookup
            .findMiddleElementFromHead1PassIteratively(
                    createNodesList(5)).get());
    assertEquals("2", MiddleElementLookup
            .findMiddleElementFromHead1PassIteratively(
                    reateNodesList(4)).get());
}
```

### 3.3 递归地一次性找到中间元素

一次性解决此问题的另一种方法是使用递归，**我们可以迭代到列表末尾来获取大小，然后在回调中，我们只需计数到大小的一半即可**。

为了在Java中做到这一点，我们将创建一个辅助类，以在执行所有递归调用期间保留列表大小和中间元素的引用：
```java
private static class MiddleAuxRecursion {
    Node middle;
    int length = 0;
}
```

现在，让我们实现递归方法：
```java
private static void findMiddleRecursively(Node node, MiddleAuxRecursion middleAux) {
    if (node == null) {
        // reached the end
        middleAux.length = middleAux.length / 2;
        return;
    }
    middleAux.length++;
    findMiddleRecursively(node.next(), middleAux);

    if (middleAux.length == 0) {
        // found the middle
        middleAux.middle = node;
    }

    middleAux.length--;
}
```

最后，让我们创建一个调用递归的方法：
```java
public static Optional<String> findMiddleElementFromHead1PassRecursively(Node head) {
    if (head == null) {
        return Optional.empty();
    }

    MiddleAuxRecursion middleAux = new MiddleAuxRecursion();
    findMiddleRecursively(head, middleAux);
    return Optional.of(middleAux.middle.data());
}
```

再次，我们可以按照与之前相同的方式进行测试：
```java
@Test
void whenFindingMiddleFromHead1PassRecursively_thenMiddleFound() {
    assertEquals("3", MiddleElementLookup
            .findMiddleElementFromHead1PassRecursively(
                    createNodesList(5)).get());
    assertEquals("2", MiddleElementLookup
            .findMiddleElementFromHead1PassRecursively(
                    createNodesList(4)).get());
}
```

## 4. 总结

在本文中，我们介绍了在Java中查找链表中间元素的问题，并展示了解决该问题的不同方法。