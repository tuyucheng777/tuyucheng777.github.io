---
layout: post
title:  使用Java清除Scanner缓冲区
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在Java中，[Scanner](https://www.baeldung.com/java-scanner)类通常用于从各种来源(包括控制台)读取输入。但是，Scanner可能会在[缓冲区](https://www.baeldung.com/java-bytebuffer)中留下换行符，从而导致后续输入读取出现问题。因此，清除Scanner缓冲区可确保输入处理一致且可预测。

**在本教程中，我们将深入研究清除Scanner缓冲区的不同Java方法**。

## 2. 使用nextLine()和hasNextLine()方法

一种简单的缓冲区清除方法涉及[nextLine()](https://www.baeldung.com/java-scanner-nextline)和[hasNextLine()](https://www.baeldung.com/java-scanner-hasnext-vs-hasnextline)方法，我们来看一个简单的例子：

```java
Scanner scanner = new Scanner("123\nHello World.");

@Test
public void givenInput_whenUsingNextLineWithHasNextLine_thenBufferCleared() {

    int number = scanner.nextInt();
    if (scanner.hasNextLine()) {
        scanner.nextLine();
    }
    String text = scanner.nextLine();

    assertEquals(123, number);
    assertEquals("Hello World", text);
}
```

在这里，我们通过创建一个带有字符串“123\\nHello World”的Scanner对象来模拟用户输入。之后，我们使用scanner.nextInt()方法读取一个整数，该整数将换行符留在缓冲区中。然后，**我们使用scanner.hasNextLine()方法检查缓冲区中是否有任何剩余输入，并通过调用scanner.nextLine()将其清除**。

清除缓冲区后，我们使用scanner.nextLine()方法读取字符串输入，现在正确读取的是“Hello World”。最后，我们使用assertEquals验证读取的整数是否为123，读取的字符串是否为“Hello World”。

## 3. 使用skip()方法

清除缓冲区的另一种方法是使用[scanner.skip("\\n")](https://www.baeldung.com/java-scanner-skip)方法，该方法会跳过缓冲区中剩余的任何换行符。让我们通过一个简单的实现来看看它是如何工作的：

```java
@Test
public void givenInput_whenUsingSkip_thenBufferCleared() {
    int number = scanner.nextInt();
    scanner.skip("\n");
    String text = scanner.nextLine();

    assertEquals(123, number);
    assertEquals("Hello World", text);
}
```

在此示例中，使用nextInt()方法读取整数后，我们使用skip("\\n")方法跳过缓冲区中剩余的换行符。这将清除缓冲区，从而允许后续的nextLine()方法正确读取“Hello World”。

## 4. 总结

总之，管理Scanner缓冲区对于Java应用程序中一致且准确的输入处理至关重要。在本教程中，我们探索了使用nextLine()和hasNextLine()清除Scanner缓冲区。