---
layout: post
title:  Java调用堆栈的最大深度是多少
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

[调用栈](https://www.baeldung.com/cs/call-stack)是Java中管理方法执行和变量作用域的关键数据结构，在处理递归函数或深度调用链时，栈深度(即栈可容纳的活动方法调用数量)是一个重要的考虑因素。

在本教程中，我们将探索确定Java调用[堆栈](https://www.baeldung.com/java-stack-heap)最大深度的技术。

## 2. 理解Java调用堆栈

**Java调用栈遵循后进先出(LIFO)结构**，调用方法时，会将一个新的栈帧推送到栈顶，其中包含参数、局部变量和返回地址等信息。方法执行完成后，其栈帧会弹出。

**分配给每个线程的总堆栈大小决定了其调用堆栈可以容纳的数据量**，默认堆栈大小因[JVM](https://www.baeldung.com/jvm-parameters)实现而异，但对于标准JVM，通常约为1MB。

我们可以使用-XX:+PrintFlagsFinal参数检查JVM的默认堆栈大小：

```shell
$ java -XX:+PrintFlagsFinal -version | grep ThreadStackSize
```

对于1MB的堆栈，假设每个堆栈帧占用大约100字节，我们可以进行大约10000到20000次方法调用，直到达到最大深度，**关键在于堆栈大小限制了调用堆栈的深度**。

## 3. Java调用栈的最大深度

下面是一个故意[溢出](https://www.baeldung.com/java-stack-overflow-error)调用堆栈以确定其最大深度的示例：

```java
public class RecursiveCallStackOverflow {
    static int depth = 0;
   
    private static void recursiveStackOverflow() {
        depth++;
        recursiveStackOverflow();
    }
    
    public static void main(String[] args) {
        try {
            recursiveStackOverflow();
        } catch (StackOverflowError e) {
            System.out.println("Maximum depth of the call stack is " + depth);
        }
    }
}
```

recursiveStackOverflow()只是简单地递增一个计数器，并递归调用自身，直到堆栈溢出。通过捕获产生的错误，我们可以打印出达到的深度。

当我们在标准JVM上测试它时，输出如下：

```shell
Maximum depth of the call stack is 21792
```

让我们使用-XX:+PrintFlagsFinal参数检查JVM的默认堆栈大小：

```shell
$ java -XX:+PrintFlagsFinal -version | grep ThreadStackSize
```

这是我们的JVM的默认线程堆栈大小：

```shell
intx ThreadStackSize = 1024 
```

默认情况下，JVM分配1MB的线程堆栈大小。

我们可以使用-Xss JVM参数为线程分配更多堆栈空间来增加最大深度：

```shell
$ java -Xss2m RecursiveCallStackOverflow
```

线程堆栈大小为2MB时，输出如下：

```java
Maximum depth of the call stack is 49522
```

堆栈大小加倍可使深度相应增加。

## 4. 总结

在本文中，我们学习了如何通过递归调用方法来获取堆栈调用的最大深度。此外，我们还了解到JVM有一个默认堆栈大小，可以通过分配更多内存空间来增加堆栈调用的深度。