---
layout: post
title:  从List中删除第一个元素
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个简短的教程中，我们将展示如何从List中删除第一个元素。

我们将为List接口的两个常见实现-ArrayList和LinkedList执行此操作。

## 2. 创建List

首先，让我们填充我们的List：

```java
@Before
public void init() {
    list.add("cat");
    list.add("dog");
    list.add("pig");
    list.add("cow");
    list.add("goat");

    linkedList.add("cat");
    linkedList.add("dog");
    linkedList.add("pig");
    linkedList.add("cow");
    linkedList.add("goat");
}
```

## 3. ArrayList

其次，让我们从ArrayList中删除第一个元素，并确保我们的列表不再包含它：

```java
@Test
public void givenList_whenRemoveFirst_thenRemoved() {
    list.remove(0);

    assertThat(list, hasSize(4));
    assertThat(list, not(contains("cat")));
}
```

如上所示，我们使用remove(index)方法删除第一个元素-**这也适用于List接口的任何实现**。

## 4. LinkedList

LinkedList也实现了remove(index)方法(以它自己的方式)，但它也有removeFirst()方法。

让我们确保它按预期工作：

```java
@Test
public void givenLinkedList_whenRemoveFirst_thenRemoved() {
    linkedList.removeFirst();

    assertThat(linkedList, hasSize(4));
    assertThat(linkedList, not(contains("cat")));
}
```

## 5. 时间复杂度

尽管这些方法看起来相似，但它们的效率不同。ArrayList的remove()方法需要O(n)时间，而LinkedList的removeFirst()方法需要O(1)时间。

这是因为ArrayList在底层使用数组，而remove()操作需要将数组的其余部分复制到开头。数组越大，需要移动的元素就越多。

与此不同，LinkedList使用指针，这意味着每个元素都指向下一个和前一个元素。

因此，删除第一个元素意味着只是更改指向第一个元素的指针。此操作始终需要相同的时间，而不取决于列表的大小。

## 6.  总结

在本文中，我们介绍了如何从List中删除第一个元素，并比较了此操作对于ArrayList和LinkedList实现的效率。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-1)上获得。