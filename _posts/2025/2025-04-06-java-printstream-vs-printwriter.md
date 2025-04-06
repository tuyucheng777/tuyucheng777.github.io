---
layout: post
title:  Java中的PrintStream与PrintWriter
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在本教程中，我们将比较PrintStream和PrintWriter Java类，本文将帮助程序员为这些类中的每一个找到合适的用例。

在深入了解内容之前，我们建议你查看我们之前的文章，其中我们演示了如何使用[PrintStream](https://www.baeldung.com/java-printstream-printf)和[PrintWriter](https://www.baeldung.com/java-write-to-file)。

## 2. PrintStream和PrintWriter的相似之处

由于PrintStream和PrintWriter共享一些功能，程序员有时很难找到这些类的合适用例。我们首先来找出它们的相似之处；然后，我们来看看它们的区别。

### 2.1 字符编码

无论何种系统，**[字符编码](https://www.baeldung.com/java-char-encoding)都允许程序操纵文本，以便跨平台一致地解释文本**。

在JDK 1.4版本之后，PrintStream类在其构造函数中包含了一个字符编码参数，这允许PrintStream类在跨平台实现中编码/解码文本。另一方面，PrintWriter从一开始就一直具有字符编码功能。

我们可以参考官方Java代码来确认：

```java
public PrintStream(OutputStream out, boolean autoFlush, String encoding) throws UnsupportedEncodingException {
    this(requireNonNull(out, "Null output stream"), autoFlush, toCharset(encoding));
}
```

类似地，PrintWriter构造函数有一个charset参数来指定用于编码目的的Charset：

```java
public PrintWriter(OutputStream out, boolean autoFlush, Charset charset) {
    this(new BufferedWriter(new OutputStreamWriter(out, charset)), autoFlush);

    // save print stream for error propagation
    if (out instanceof java.io.PrintStream) {
        psOut = (PrintStream) out;
    }
}
```

**如果没有为这些类中的任何一个提供字符编码，它们将使用默认的平台编码**。

### 2.2 写入文件

要将文本写入文件，我们可以将String或File实例传递给相应的构造函数。另外，我们可以传递字符集进行字符编码。

例如，我们将引用具有单个File参数的构造函数。在这种情况下，字符编码将默认为平台：

```java
public PrintStream(File file) throws FileNotFoundException {
    this(false, new FileOutputStream(file));
}
```

类似地，PrintWriter类有一个构造函数来指定要写入的文件：

```java
public PrintWriter(File file) throws FileNotFoundException {
    this(new BufferedWriter(new OutputStreamWriter(new FileOutputStream(file))), false);
}
```

我们可以看到，**这两个类都提供了写入文件的功能，但是，它们的实现因使用不同的流父类而有所不同**，我们将在本文的差异部分深入探讨为什么会出现这种情况。

## 3. PrintStream和PrintWriter的区别

在上一节中，我们展示了PrintStream和PrintWriter具有一些可能适合我们情况的共同功能。**尽管如此，尽管我们可以用这些类做同样的事情，但它们的实现却各不相同**，这让我们需要评估哪个类更适合。

现在，让我们看看PrintStream和PrintWriter之间的区别。

### 3.1 数据处理

在上一节中，我们展示了这两个类如何写入文件，让我们看看它们的实现有何不同。

**对于PrintStream，它是OutputStream的子类，在Java中定义为字节流。换句话说，数据是逐字节处理的。另一方面，PrintWriter是一个字符流，它一次处理一个字符，并使用Unicode自动转换我们指定的每个字符集**。

我们将在两种不同的情况下展示这些实现中的每一个。

### 3.2 处理非文本数据

**由于这两个类处理数据的方式不同，因此我们可以在处理非文本文件时进行区分**。在此示例中，我们将使用png文件读取数据，然后在将其内容写入每个类的另一个文件后查看差异：

```java
public class DataStream {

    public static void main (String[] args) throws IOException {
        FileInputStream inputStream = new FileInputStream("image.png");
        PrintStream printStream = new PrintStream("ps.png");

        int b;
        while ((b = inputStream.read()) != -1) {
            printStream.write(b);
        }
        printStream.close();

        FileReader reader = new FileReader("image.png");
        PrintWriter writer = new PrintWriter("pw.png");

        int c;
        while ((c = reader.read()) != -1) {
            writer.write(c);
        }
        writer.close();
    }
}
```

在这个例子中，我们使用FileInputStream和FileReader来读取图像的内容。然后，我们将数据写入不同的输出文件。

因此，ps.png和pw.png文件将根据流处理其内容的方式包含数据。**我们知道PrintStream通过一次读取一个字节来处理数据，因此，生成的文件包含与原始文件相同的原始数据**。

**与PrintStream类不同，PrintWriter将数据解释为字符，这会导致文件损坏，系统无法理解其内容**。或者，我们可以将pw.png的扩展名更改为pw.txt，并检查PrintWriter如何尝试将图像的原始数据转换为难以辨认的符号。

### 3.3 处理文本数据

**现在，让我们看一个示例，其中我们使用OutputStream(PrintStream的父类)来演示在写入文件时如何处理字符串**：

```java
public class PrintStreamWriter {
    public static void main (String[] args) throws IOException {
        OutputStream out = new FileOutputStream("TestFile.txt");
        out.write("foobar");
        out.flush();
    }
}
```

**上面的代码将无法编译，因为OutputStream不知道如何处理字符串。要成功写入文件，输入数据必须是原始字节序列**。以下更改将使我们的代码成功写入文件：

```java
out.write("foobar".getBytes());
```

回到PrintStream，虽然这个类是OutputStream的子类，但**Java在内部调用了getBytes()方法**。这使得PrintStream在调用print方法时可以接收字符串，让我们看一个例子：

```java
public class PrintStreamWriter {
    public static void main (String[] args) throws IOException {
        PrintStream out = new PrintStream("TestFile.txt");
        out.print("Hello, world!");
        out.flush();
    }
}
```

现在，因为PrintWriter知道如何处理字符串，所以我们调用print方法传递字符串输入。但是，在这种情况下，**Java并没有将字符串转换为字节，而是在内部将流中的每个字符转换为其对应的Unicode编码**：

```java
public class PrintStreamWriter {
    public static void main (String[] args) throws IOException {
        PrintWriter out = new PrintWriter("TestFile.txt");
        out.print("Hello, world!");
        out.flush();
    }
}
```

基于这些类在内部处理文本数据的方式，**字符流类(如PrintWriter)在对文本进行[I/O操作](https://www.baeldung.com/java-io)时可以更好地处理这些内容**。此外，在本地字符集的编码过程中将数据翻译成Unicode使应用程序的[国际化](https://www.baeldung.com/java-8-localization)更简单。

### 3.4 刷新

在我们前面的示例中，请注意我们必须如何显式调用flush方法。根据Java文档，**这个过程在这两个类之间的工作方式不同**。

对于PrintStream，我们可以指定仅在写入字节数组、调用println方法或写入换行符时自动刷新。但是，PrintWriter也可以自动刷新，但只有当我们调用println、printf或format方法时才可以。

这种区别很难证明，因为文档提到在上述情况下会发生刷新，但没有提到何时不会发生刷新。因此，**我们可以演示自动刷新在这两个类中的工作原理，但我们不能保证它会按预期运行**。

在此示例中，我们将启用自动刷新功能并在末尾写入一个带有换行符的字符串：

```java
public class AutoFlushExample {
    public static void main (String[] args) throws IOException {
        PrintStream printStream = new PrintStream(new FileOutputStream("autoFlushPrintStream.txt"), true);
        printStream.write("Hello, world!\n".getBytes());
        printStream.close();

        PrintWriter printWriter = new PrintWriter(new FileOutputStream("autoFlushPrintWriter.txt"), true);
        printWriter.print("Hello, world!");
        printWriter.close();
    }
}
```

**由于我们启用了自动刷新功能，因此可以保证文件autoFlushPrintStream.txt将包含写入文件的内容**。此外，我们使用包含换行符的字符串调用write方法以强制刷新。

但是，我们希望看到autoFlushPrintWriter.txt文件为空，尽管这不能保证。毕竟，刷新可能发生在程序执行期间。

如果我们想在使用PrintWriter时强制刷新，则代码必须满足我们上面提到的所有要求，或者我们可以添加一行代码来显式刷新writer：

```java
printWriter.flush();
```

## 4. 总结

在本文中，我们比较了两个数据流类PrintStream和PrintWriter。首先，我们研究了它们的相似性和使用本地字符集的能力。此外，我们还介绍了如何读取和写入外部文件的示例。虽然我们可以用这两个类实现类似的功能，但在了解了差异之后，我们证明了每个类在不同场景中的表现更好。

例如，在写入所有类型的数据时，PrintStream对我们更有益，因为PrintStream处理的是原始字节。另一方面，作为字符流的PrintWriter最适合在执行I/O操作时处理文本。此外，由于其内部格式为Unicode，因此它有助于实现复杂的软件实现，例如国际化。最后，我们比较了这两个类中刷新实现的不同之处。