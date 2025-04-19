---
layout: post
title:  在Java中创建自定义链表数据结构
category: algorithms
copyright: algorithms
excerpt: 链表
---

## 1. 概述

与[数组](https://www.baeldung.com/java-arrays-guide)(分配连续的内存块进行顺序存储)不同，**链表将其元素分布在非连续的内存位置，每个节点都引用下一个节点**。链表的结构允许动态调整大小，从而提高插入效率。

在本教程中，我们将学习如何在Java中实现自定义单链表，并具有插入、删除、检索和计数元素的功能。

值得注意的是，由于Java标准库提供了[LinkedList](https://www.baeldung.com/java-linkedlist)实现，我们的自定义实现纯粹是为了教育目的。

## 2. 理解链表

链表是节点的集合，每个节点存储一个值以及指向序列中下一个节点的引用，**链表的最小单位是单个[节点](https://www.baeldung.com/cs/linked-list-data-structure)**。

在典型的单链表中，第一个节点是头，最后一个节点是尾。尾对下一个节点的引用始终为null，标志着链表的结束。

链表的常见类型有三种：

- 单链表：每个节点仅指向下一个节点
- 双链表：每个节点都包含对下一个节点和前一个节点的引用
- 循环链表：最后一个节点的引用指向链表头部，形成一个循环，它可以是单链表，也可以是双链表。

## 3. 创建节点

让我们创建一个名为Node的类：

```java
class Node<T> {
    T value;
    Node<T> next;

    public Node(T value) {
        this.value = value; 
        this.next = null;
    }
}
```

上面的类表示链表中的单个元素，它有两个字段：

- value：存储此节点的数据
- next：保存对列表中下一个节点的引用

**通过使用[泛型](https://www.baeldung.com/java-generics)(<T\>)，我们确保我们的链表可以存储任何指定类型的元素**。

## 4. 创建和链接节点

接下来我们看看节点是如何链接在一起的：

```java
Node<Integer> node0 = new Node<>(1);
Node<Integer> node1 = new Node<>(2);
Node<Integer> node2 = new Node<>(3);
node0.next = node1;
node1.next = node2;
```

这里，我们创建3个Integer类型的节点，并使用next引用将它们链接在一起。然后，让我们用断言来验证结构：

```java
assertEquals(1, node0.value);
assertEquals(2, node0.next.value);
assertEquals(3, node0.next.next.value);
```

这些断言确认每个节点正确引用序列中的下一个节点。

值得注意的是，尝试链接不同类型的节点会导致编译错误。

## 5. 实现链表操作

在上一节中，我们实现了一个基本的Node类，让我们在此基础上创建一个自定义的单链表。

### 5.1 定义CustomLinkedList类

首先，让我们创建一个名为CustomLinkedList的类：

```java
public class CustomLinkedList<T> {
    private Node<T> head;
    private Node<T> tail;
    private int size;

    public CustomLinkedList() {
        head = null;
        tail = null;
        size = 0;
    }

    // ...
}
```

在上面的代码中，该类保存对头节点和尾节点的引用。

当我们创建一个空的CustomLinkedList对象时，头和尾会被设置为null。size用来跟踪节点的数量，默认设置为0。

### 5.2 在尾部插入元素

由于链表可以扩展，因此让我们向CustomLinkedList添加一个方法，用于在列表末尾添加一个新节点：

```java
public void insertTail(T value) {
    if (size == 0) {
        head = tail = new Node<>(value);
    } else {
        tail.next = new Node<>(value);
        tail = tail.next;
    }
    size++;
}
```

上述方法接收我们想要添加的值作为参数，如果列表为空，则head和tail都指向新节点。

否则，新节点将链接到现有的尾部，并更新尾部。

### 5.3 在头部插入元素

此外，让我们实现一个在列表开头插入节点的方法：

```java
public void insertHead(T value) {
    Node<T> newNode = new Node<>(value);
    newNode.next = head;
    head = newNode;

    if (size == 0) {
        tail = newNode;
    }

    size++;
}
```

这里，新节点指向当前的head，使其成为新的head。如果链表为空，则更新tail指向新节点。

### 5.4 检索元素

接下来，让我们定义一个方法来通过索引检索节点的值：

```java
public T get(int index) {
    if (index < 0 || index >= size) {
        throw new IndexOutOfBoundsException("Index out of bounds");
    }

    Node<T> current = head;
    for (int i = 0; i < index; i++) {
        current = current.next;
    }

    return current.value;
}
```

在上面的代码中，我们验证了索引以防止越界错误。接下来，我们遍历列表，直到到达指定的索引。最后，我们返回存储在该节点的值。

### 5.5 删除头节点

要删除第一个元素，我们只需更新head以指向下一个节点：

```java
public void removeHead() {
    if (head == null) {
        return;
    }
    head = head.next;
    if (head == null) {
        tail = null;
    }
    size--;
}
```

这里，我们更新head指向下一个节点，实际上就是删除了旧的head节点。如果链表为空，则tail也设置为null。

### 5.6 通过索引删除节点

为了更加灵活，让我们实现一种方法来删除指定索引处的节点：

```java
public void removeAtIndex(int index) {
    if (index < 0 || index >= size) {
        throw new IndexOutOfBoundsException("Index out of bounds");
    }

    if (index == 0) {
        removeHead();
        return;
    }

    Node<T> current = head;
    for (int i = 0; i < index - 1; i++) {
        current = current.next;
    }

    if (current.next == tail) {
        tail = current;
    }

    current.next = current.next.next;
    size--;
}
```

首先，如果索引为0，我们使用removeHead()方法移除头节点。接下来，我们遍历到目标节点之前的节点，然后，我们更新next指针以跳过被移除的节点。

如果我们删除最后一个元素，则尾部会被更新。

### 5.7 返回列表的大小

最后，我们添加一个方法来返回链表的大小：

```java
public int size() {
    return size;
}
```

在这里，我们返回在删除或添加节点时更新的size字段。

## 6. 总结

在本文中，我们学习了如何创建一个自定义链表，该链表与Java内置的LinkedList类似。此外，我们还实现了插入、检索和删除方法来管理自定义链表中的元素。