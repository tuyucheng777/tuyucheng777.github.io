---
layout: post
title:  将OutputStream转换为InputStream
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

InputStream和[OutputStream](https://www.baeldung.com/java-outputstream)是Java IO中的两个基本类。有时，我们需要在这两种流类型之间进行转换。在[之前的教程](https://www.baeldung.com/java-inputstream-to-outputstream)中，我们讨论了如何将InputStream写入OutputStream。

在本快速教程中，我们将探索如何将OutputStream转换为InputStream。

## 2. 问题介绍

有时，需要将OutputStream转换为InputStream，这种转换在很多情况下都很有用，例如**当我们需要读取已写入OutputStream的数据时**。

在本文中，我们将探讨执行此转换的两种不同方法：

- 使用字节数组
- 使用管道

为简单起见，我们将在示例中使用[ByteArrayOutputStream](https://www.baeldung.com/java-outputstream#2-bytearrayoutputstream)作为OutputStream类型。此外，我们将使用单元测试断言来验证是否可以从转换后的InputStream对象中读取预期数据。

接下来让我们看看它们的实际效果。

## 3. 使用字节数组

当我们思考这个问题时，最直接的方法可能是：

- 步骤1：从给定的OutputStream读取数据，并将数据保存在缓冲区中，例如字节数组中
- 步骤2：[使用字节数组创建InputStream](https://www.baeldung.com/convert-byte-array-to-input-stream)

接下来，让我们将这个想法作为测试来实现，并检查它是否按预期工作：

```java
@Test
void whenUsingByteArray_thenGetExpectedInputStream() throws IOException {
    String content = "I'm an important message.";
    try (ByteArrayOutputStream out = new ByteArrayOutputStream()) {
        out.write(content.getBytes());
        try (ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray())) {
            String inContent = new String(in.readAllBytes());

            assertEquals(content, inContent);
        }
    }
}
```

首先，我们准备一个OutputStream对象(out)并向其中写入一个字符串(content)。接下来，**我们通过调用out.toByteArray()从OutputStream中获取字节数组形式的数据**，并从该数组创建一个InputStream。

如果我们运行测试，它就会通过，因此转换成功。

值得一提的是，**我们使用了[try-with-resources](https://www.baeldung.com/java-try-with-resources)来确保在读取和写入操作后关闭InputStream和OutputStream**。此外，我们的try块没有catch块，因为我们已经声明测试方法会抛出IOException。

这种方法简单易行，但是缺点是它需要将整个输出存储在内存中，然后才能将其读回作为输入。换句话说，**如果输出非常大，可能会导致大量内存消耗**，并可能导致[OutOfMemoryError](https://www.baeldung.com/java-gc-overhead-limit-exceeded)。

## 4. 通过管道

我们经常一起使用[PipedOutputStream](https://www.baeldung.com/java-outputstream#5-pipedoutputstream)和PipedInputStream类来将数据从OutputStream传递到InputStream。因此，**我们可以先连接PipedOutputStream和PipedInputStream，以便PipedInputStream可以读取来自PipedOutputStream的数据**。接下来，我们可以将给定的OutputStream的数据写入PipedOutputStream。然后，在另一端，我们可以从PipedInputStream读取数据。

接下来，让我们将其作为单元测试来实现：

```java
@Test
void whenUsingPipeStream_thenGetExpectedInputStream() throws IOException {
    String content = "I'm going through the pipe.";

    ByteArrayOutputStream originOut = new ByteArrayOutputStream();
    originOut.write(content.getBytes());

    //connect the pipe
    PipedInputStream in = new PipedInputStream();
    PipedOutputStream out = new PipedOutputStream(in);

    try (in) {
        new Thread(() -> {
            try (out) {
                originOut.writeTo(out);
            } catch (IOException iox) {
                // ...
            }
        }).start();

        String inContent = new String(in.readAllBytes());
        assertEquals(content, inContent);
    }
}
```

如上面的代码所示，首先我们准备好OutputStream(originOut)，然后我们创建PipedInputStream(in)和PipedOutputStream(out)，并将它们连接起来，这样就建立了一个管道。

然后，我们使用ByteArrayOutputStream.writeTo()方法将数据从给定的OutputStream转发到PipedOutputStream。我们应该注意，我们创建了一个新线程来将数据写入PipedOutputStream，这是因为**不建议在同一个线程中同时使用PipedInputStream和PipedOutputStream对象，这可能会导致死锁**。

最后，如果我们执行它，测试就会通过。因此，OutputStream已成功转换为PipedInputStream。

## 5. 总结

在本文中，我们探讨了将OutputStream转换为InputStream的两种方法：

- 字节数组作为缓冲区：这很简单，但是它有OutOfMemoryError的潜在风险
- 使用管道：将输出写入PipedOutputStream使数据流向PipedInputStream