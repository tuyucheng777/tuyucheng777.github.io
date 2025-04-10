---
layout: post
title:  在Java中反转链表
category: algorithms
copyright: algorithms
excerpt: 链表
---

## 1. 简介

在本教程中，我们将用Java实现两种[链表反转算法](https://www.baeldung.com/cs/reverse-linked-list)。

## 2. 链表数据结构

链表是一种线性数据结构，其中每个元素中的指针决定顺序。**链表的每个元素都包含一个数据字段来存储列表数据，以及一个指针字段来指向序列中的下一个元素**。此外，我们可以使用头指针指向链表的起始元素：

![](/assets/images/2025/algorithms/javareverselinkedlist01.png)

当我们反转链表之后，head会指向原链表的最后一个元素，而每个元素的指针会指向原链表的前一个元素：

![](/assets/images/2025/algorithms/javareverselinkedlist02.png)

在Java中，我们有一个[LinkedList](https://www.baeldung.com/java-linkedlist)类，它提供了List和Deque接口的双向链表实现。但是，在本教程中，我们将使用通用的单向链表数据结构。

让我们首先从一个ListNode类开始，来表示链表的一个元素：
```java
public class ListNode {

    private int data;
    private ListNode next;

    ListNode(int data) {
        this.data = data;
        this.next = null;
    }

    // standard getters and setters
}
```

ListNode类有两个字段：

- 表示元素数据的整数值
- 指向下一个元素的指针/引用

一个链表可能包含多个ListNode对象，例如，我们可以用循环构造上面的示例链表：
```java
ListNode constructLinkedList() {
    ListNode head = null;
    ListNode tail = null;
    for (int i = 1; i <= 5; i++) {
        ListNode node = new ListNode(i);
        if (head == null) {
            head = node;
        } else {
            tail.setNext(node);
        }
        tail = node;
    }
    return head;
}
```

## 3. 迭代算法实现

让我们用Java实现[迭代算法](https://www.baeldung.com/cs/reverse-linked-list#iterative)：
```java
ListNode reverseList(ListNode head) {
    ListNode previous = null;
    ListNode current = head;
    while (current != null) {
        ListNode nextElement = current.getNext();
        current.setNext(previous);
        previous = current;
        current = nextElement;
    }
    return previous;
}
```

在这个迭代算法中，我们使用两个ListNode变量previous和current来表示链表中的两个相邻元素。对于每次迭代，我们将这两个元素反转，然后移至接下来的两个元素。

最终，current指针将为null，而previous指针将是旧链表的最后一个元素。因此，previous也是反向链表的新头指针，我们从方法中返回它。

我们可以用一个简单的单元测试来验证这个迭代实现：
```java
@Test
void givenLinkedList_whenIterativeReverse_thenOutputCorrectResult() {
    ListNode head = constructLinkedList();
    ListNode node = head;
    for (int i = 1; i <= 5; i++) {
        assertNotNull(node);
        assertEquals(i, node.getData());
        node = node.getNext();
    }

    LinkedListReversal reversal = new LinkedListReversal();
    node = reversal.reverseList(head);

    for (int i = 5; i >= 1; i--) {
        assertNotNull(node);
        assertEquals(i, node.getData());
        node = node.getNext();
    }
}
```

在此单元测试中，我们首先构建一个包含5个节点的示例链表。此外，我们验证链表中的每个节点是否包含正确的数据值。然后，我们调用迭代函数来反转链表。最后，我们检查反转后的链表以确保数据按预期反转。

## 4. 递归算法实现

现在，让我们用Java实现[递归算法](https://www.baeldung.com/cs/reverse-linked-list#recursive)：
```java
ListNode reverseListRecursive(ListNode head) {
    if (head == null) {
        return null;
    }
    if (head.getNext() == null) {
        return head;
    }
    ListNode node = reverseListRecursive(head.getNext());
    head.getNext().setNext(head);
    head.setNext(null);
    return node;
}
```

在reverseListRecursive函数中，我们递归访问链表中的每个元素，直到到达最后一个元素。最后一个元素将成为反向链表的新头元素，此外，我们将访问过的元素附加到部分反向链表的末尾。

类似地，我们可以通过一个简单的单元测试来验证这个递归实现：
```java
@Test
void givenLinkedList_whenRecursiveReverse_thenOutputCorrectResult() {
    ListNode head = constructLinkedList();
    ListNode node = head;
    for (int i = 1; i <= 5; i++) {
        assertNotNull(node);
        assertEquals(i, node.getData());
        node = node.getNext();
    }

    LinkedListReversal reversal = new LinkedListReversal();
    node = reversal.reverseListRecursive(head);

    for (int i = 5; i >= 1; i--) {
        assertNotNull(node);
        assertEquals(i, node.getData());
        node = node.getNext();
    }
}
```

## 5. 总结

在本教程中，我们实现了两种算法来反转链表。