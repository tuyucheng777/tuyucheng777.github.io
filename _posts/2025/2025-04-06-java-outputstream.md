---
layout: post
title:  Java OutputStream指南
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将探索有关Java类OutputStream的详细信息。OutputStream是一个抽象类，**它是所有表示字节输出流的类的超类**。

我们将在后面的内容中详细探讨“输出”和“流”这些词的含义。

## 2. Java IO简介

**OutputStream是Java IO API的一部分**，它定义了在Java中执行I/O操作所需的类。这些都打包在java.io命名空间中，这是自1.0版本以来Java中可用的核心包之一。

从Java 1.4开始，还有Java NIO打包在命名空间java.nio中，它支持非阻塞输入和输出操作。不过，本文的重点是作为Java IO一部分的ObjectStream。

可以在[此处](https://docs.oracle.com/javase/8/docs/technotes/guides/io/index.html)找到与Java IO和Java NIO相关的详细信息。

### 2.1 输入和输出

**Java IO基本上提供了一种从源读取数据并将数据写入目标的机制**，输入代表来源而输出代表这里的目的地。

这些源和目标可以是从文件、管道到网络连接的任何内容。

### 2.2 流

Java IO提供了流的概念，**它基本上表示连续的数据流**。流可以支持许多不同类型的数据，如字节、字符、对象等。

此外，与源或目标的连接是流所代表的。因此，它们分别以InputStream或OutputStream的形式出现。

## 3、OutputStream的接口

OutputStream实现了许多接口，这些接口为其子类提供了一些独特的特性，让我们快速浏览一下。

### 3.1 Closeable

接口Closeable提供了一个名为close()的方法，它处理关闭数据源或数据目标。OutputStream的每个实现都必须提供此方法的实现，它们可以在此处执行操作以释放资源。

### 3.2 AutoCloseable

接口AutoCloseable也提供了一个名为close()的方法，其行为与Closeable中的方法类似。但是，在这种情况下，**方法close()会在退出try-with-resource块时自动调用**。

有关try-with-resource的更多详细信息，请参见[此处](https://www.baeldung.com/java-try-with-resources)。

### 3.3 Flushable

接口Flushable提供了一个名为flush()的方法，用于将数据刷新到目的地。

OutputStream的特定实现可能会选择缓冲先前写入的字节以进行优化，但**对flush()的调用会使其立即写入目标**。

## 4. OutputStream中的方法

OutputStream有几个方法，每个实现类都必须为它们各自的数据类型实现这些方法。

这些与从Closeable和Flushable接口继承的close()和flush()方法不同。

### 4.1 write(int b)

我们可以使用此方法将一个特定字节写入OutputStream，由于参数“int”包含四个字节，因此根据约定，仅写入第一个低位字节，其余3个高位字节将被忽略：

```java
public static void fileOutputStreamByteSingle(String file, String data) throws IOException {
    byte[] bytes = data.getBytes();
    try (OutputStream out = new FileOutputStream(file)) {
        out.write(bytes[6]);
    }
}
```

如果我们使用“Hello World!”作为数据调用此方法，我们得到的结果是一个包含以下文本的文件：

```text
W
```

正如我们所见，这是索引为6的字符串的第7个字符。

### 4.2 write(byte[] b，int off，int length)

**write()方法的这个重载版本用于将字节数组的子序列写入OutputStream**。

它可以从参数指定的字节数组中写入“length”字节数，从“off”确定的偏移量开始到OutputStream：

```java
public static void fileOutputStreamByteSubSequence(
  String file, String data) throws IOException {
    byte[] bytes = data.getBytes();
    try (OutputStream out = new FileOutputStream(file)) {
        out.write(bytes, 6, 5);
    }
}
```

如果我们现在使用与以前相同的数据调用此方法，我们将在输出文件中得到以下文本：

```text
World
```

这是我们数据的子字符串，从索引5开始，由5个字符组成。

### 4.3 write(byte[] b)

这是write()方法的另一个重载版本，它可以按照参数的指定将整个字节数组写入OutputStream。

这与调用write(b, 0, b.length)具有相同的效果：

```java
public static void fileOutputStreamByteSequence(String file, String data) throws IOException {
    byte[] bytes = data.getBytes();
    try (OutputStream out = new FileOutputStream(file)) {
        out.write(bytes);
    }
}
```

当我们使用相同的数据调用这个方法时，我们的输出文件中会有整个字符串：

```text
Hello World!
```

## 5. OutputStream的直接子类

现在我们将讨论OutputStream的一些直接已知子类，它们分别表示它们定义的OutputStream的特定数据类型。

除了实现从OutputStream继承的方法之外，它们还定义了自己的方法。

我们不会深入讨论这些子类的细节。

### 5.1 FileOutputStream

顾名思义，FileOutputStream是将数据写入File的OutputStream。FileOutputStream与任何其他OutputStream一样，可以写入原始字节流。

在上一节中，我们已经研究了FileOutputStream中的不同方法。

### 5.2 ByteArrayOutputStream

ByteArrayOutputStream是OutputStream的一种实现，可以将数据写入字节数组。随着ByteArrayOutputStream向缓冲区写入数据，缓冲区会不断增长。

我们可以将缓冲区的默认初始大小保持为32字节，或者使用可用的构造函数之一设置特定大小。

这里需要注意的重要一点是方法close()实际上没有任何效果，即使在调用close()之后，也可以安全地调用ByteArrayOutputStream中的其他方法。

### 5.3 FilterOutputStream

OutputStream主要将字节流写入目标，但它也可以在写入之前转换数据。FilterOutputStream表示执行特定数据转换的所有此类类的超类，FilterOutputStream始终使用现有的OutputStream构造。

FilterOutputStream的一些示例是BufferedOutputStream、CheckedOutputStream、CipherOutputStream、DataOutputStream、DeflaterOutputStream、DigestOutputStream、InflaterOutputStream、PrintStream。

### 5.4 ObjectOutputStream

ObjectOutputStream可以将原始数据类型和Java对象图写入目标，我们可以使用现有的OutputStream构造一个ObjectOutputStream来写入特定的目的地，如File。

请注意，对象必须实现Serializable才能让ObjectOutputStream将其写入目标，你可以在[此处](https://www.baeldung.com/java-serialization)找到有关Java序列化的更多详细信息。

### 5.5 PipedOutputStream

PipedOutputStream可用于创建通信管道，PipedOutputStream可以写入数据，连接的PipedInputStream可以读取数据。

PipedOutputStream具有一个构造函数，用于将其与PipedInputStream连接起来。或者，我们可以稍后使用PipedOutputStream中提供的称为connect()的方法来完成此操作。

## 6. OutputStream缓冲

输入和输出操作通常涉及相对昂贵的操作，如磁盘访问、网络活动等，经常执行这些操作会降低程序的效率。

在Java中，我们有“缓冲数据流”来处理这些情况。BufferedOutputStream将数据写入缓冲区，而当缓冲区已满或调用flush()方法时，该缓冲区将刷新到目标。

BufferedOutputStream扩展了前面讨论过的FilterOutputStream并包装了现有的OutputStream以写入目的地：

```java
public static void bufferedOutputStream(String file, String ...data) throws IOException {
    try (BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(file))) {
        for(String s : data) {
            out.write(s.getBytes());
            out.write(" ".getBytes());
        }
    }
}
```

需要注意的关键点是，每个数据参数对write()的每次调用都只会写入缓冲区，而不会导致对文件进行潜在的昂贵调用。

在上面的例子中，如果我们用“Hello”、“World!”作为数据调用此方法，则仅当代码退出try-with-resources块时才会将数据写入文件，该块会调用BufferedOutputStream上的close()方法。

这将生成一个包含以下文本的输出文件：

```text
Hello World!
```

## 7. 使用OutputStreamWriter写入文本

如前所述，字节流表示可能是一堆文本字符的原始数据，现在我们可以获取字符数组并自己执行到字节数组的转换：

```java
byte[] bytes = data.getBytes();
```

Java提供了方便的类来弥补这一差距，对于OutputStream的情况，此类是OutputStreamWriter，**OutputStreamWriter包装了一个OutputStream并可以直接将字符写入所需的目的地**。

我们还可以选择为OutputStreamWriter提供编码字符集：

```java
public static void outputStreamWriter(String file, String data) throws IOException {
    try (OutputStream out = new FileOutputStream(file);
         Writer writer = new OutputStreamWriter(out,"UTF-8")) {
        writer.write(data);
    }
}
```

现在我们可以看到，在使用FileOutputStream之前，我们不必执行字符数组到字节数组的转换，OutputStreamWriter可以方便地为我们完成这一操作。

当我们用“Hello World!”之类的数据调用上述方法时，会产生一个包含以下文本的文件：

```text
Hello World!
```

## 8. 总结

在本文中，我们讨论了Java抽象类OutputStream，我们介绍了它实现的接口和它提供的方法。

然后我们讨论了Java中可用的OutputStream的一些子类，最后我们讨论了缓冲和字符流。