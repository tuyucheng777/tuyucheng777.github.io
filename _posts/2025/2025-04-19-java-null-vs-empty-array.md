---
layout: post
title:  Java中null和空数组的区别
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在本教程中，我们将探讨Java中null和空[数组](https://baeldung.com/java-arrays-guide)之间的区别，虽然它们听起来很相似，但null和空数组的行为和用途截然不同，这对于正确处理至关重要。

让我们探索一下它们的工作原理以及它们为何重要。

## 2. Java中的null数组

**Java中的空数组表示该数组引用未指向内存中的任何对象，除非我们明确赋值，否则Java默认将引用变量(包括数组)[初始化](https://baeldung.com/java-initialize-array)为null**。

如果我们尝试访问或操作null数组，则会触发NullPointerException，这是一个常见错误，表示尝试使用未初始化的对象引用：

```java
@Test
public void givenNullArray_whenAccessLength_thenThrowsNullPointerException() {
    int[] nullArray = null;
    assertThrows(NullPointerException.class, () -> {
        int length = nullArray.length; 
    });
}
```

在上面的测试用例中，我们尝试访问一个null数组的长度，结果抛出了NullPointerException异常。测试用例执行顺利，验证NullPointerException确实被抛出了。

正确处理null数组通常涉及在执行任何操作之前[检查是否为null](https://baeldung.com/java-array-check-null-empty)，以避免运行时异常。

## 3. Java中的空数组

**Java中的空数组是指已实例化但包含0个元素的数组**，这意味着它是一个有效的数组对象，可以用于操作，尽管它不包含任何值。**实例化空数组时，Java会为数组结构分配内存，但不存储任何元素**。

值得注意的是，当我们创建一个非空数组而没有为其元素指定值时，它们默认为0值-整数数组为0，布尔数组为false，对象数组为null：

```java
@Test
public void givenEmptyArray_whenCheckLength_thenReturnsZero() {
    int[] emptyArray = new int[0];
    assertEquals(0, emptyArray.length);
}
```

上述测试用例成功执行，表明空数组的长度为0，并且在访问时不会引起任何异常。

空数组通常用于稍后初始化具有固定大小的数组或表示当前不存在任何元素。

## 4. 总结

在本文中，我们研究了Java中null和空数组之间的区别。null数组表示该数组引用未指向任何对象，如果在访问时未进行适当的null检查，则可能导致NullPointerException错误。另一方面，空数组是一个有效的、实例化的数组，没有元素，其长度为0，可以实现安全的操作。