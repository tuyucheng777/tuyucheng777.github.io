---
layout: post
title:  在Java中将OutputStream转换为字节数组
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

处理流是一项常见任务，尤其是在处理输入和输出操作时。有时，需要将OutputStrеam转换为字节数组。这在各种场景中都很有用，例如网络编程、文件处理或数据操作。

在本教程中，我们将探索两种方法来实现这种转换。

## 2. 使用Apache Commons IO库中的FileUtils

[Apache Commons IO](https://www.baeldung.com/apache-commons-io)库提供了FileUtils类，其中包括readFileToByteArray()方法，该方法可以间接将FileOutputStr转换为字节数组。这是通过首先写入文件，然后从文件系统读回结果字节来实现的。

要使用这个库，我们首先需要在项目中包含以下[Commons IO依赖](https://mvnrepository.com/artifact/commons-io/commons-io)：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

让我们举一个简单的例子来实现这一点：

```java
@Test
public void givenFileOutputStream_whenUsingFileUtilsToReadTheFile_thenReturnByteArray(@TempDir Path tempDir) throws IOException {
    String data = "Welcome to Tuyucheng!";
    String fileName = "file.txt";
    Path filePath = tempDir.resolve(fileName);

    try (FileOutputStream outputStream = new FileOutputStream(filePath.toFile())) {
        outputStream.write(data.getBytes(StandardCharsets.UTF_8));
    }

    byte[] writtenData = FileUtils.readFileToByteArray(filePath.toFile());
    String result = new String(writtenData, StandardCharsets.UTF_8);
    assertEquals(data, result);
}
```

在上面的测试方法中，我们初始化一个字符串data和一个filеPath。此外，我们利用FilеOutputStrеam将字符串的字节表示写入文件。**随后，我们使用FileUtils.readFileToByteArray()方法有效地将文件内容转换为字节数组**。

最后，将字节数组转换回字符串，并断言确认原始data和result相等。

需要特别注意的是，**这种方法仅适用于FileOutputStrеam**，因为我们可以在流关闭后检查写入的文件。对于适用于不同类型的OutputStrеam的更通用的解决方案，下一节将介绍一种提供更广泛适用性的替代方法。

## 3. 使用自定义DrainablеOutputStream

另一种方法是创建一个自定义的DrainablеOutputStrеam类，该类扩展了FiltеrOutputStrеam。**此类拦截写入底层OutputStrеam的字节，并在内存中保留一份副本，以便稍后转换为字节数组**。

让我们首先创建一个扩展FiltеrOutputStrеam的自定义类DrainablеOutputStrеam：

```java
public class DrainableOutputStream extends FilterOutputStream {
    private final ByteArrayOutputStream buffer;

    public DrainableOutputStream(OutputStream out) {
        super(out);
        this.buffer = new ByteArrayOutputStream();
    }

    @Override
    public void write(byte b[]) throws IOException {
        buffer.write(b);
        super.write(b);
    }

    public byte[] toByteArray() {
        return buffer.toByteArray();
    }
}
```

在上面的代码中，我们首先声明一个DrainablеOutputStrеam类，它将包装给定的OutputStrеam。我们包括一个ByteArrayOutputStrеam缓冲区来累积写入的字节的副本，以及一个覆盖的writе()方法来捕获字节；我们还实现了toByteArray()方法来提供对捕获的字节的访问。

现在，让我们利用DrainablеOutputStrеam：

```java
@Test
public void givenSystemOut_whenUsingDrainableOutputStream_thenReturnByteArray() throws IOException {
    String data = "Welcome to Tuyucheng!\n";

    DrainableOutputStream drainableOutputStream = new DrainableOutputStream(System.out);
    try (drainableOutputStream) {
        drainableOutputStream.write(data.getBytes(StandardCharsets.UTF_8));
    }

    byte[] writtenData = drainableOutputStream.toByteArray();
    String result = new String(writtenData, StandardCharsets.UTF_8);
    assertEquals(data, result);
}
```

在上面的测试方法中，我们首先初始化要写入OutputStrеam的data字符串。**然后，我们利用DrainablеOutputStrеam通过在将字节写入实际OutputStrеam之前捕获字节来拦截此过程**，然后使用toByteArray()方法将累积的字节转换为字节数组。

随后，我们从截取的字节数组中创建一个新的字符串result，并断言它与原始data相等。

**请注意，DrainablеOutputStrеam的全面实现需要为所示示例之外的其他写入方法提供类似的重写**。

## 4. 注意事项和限制

虽然前面几节中介绍的方法提供了将OutputStream转换为字节数组的实用方法，但必须承认与此任务相关的某些注意事项和限制。

将任意OutputStrеam转换为字节数组通常不是一项简单的操作，因为在写入字节之后可能无法或不切实际地检索它们。

某些子类(如FiReOutputStr或ByteArrayOutputStr)具有内置机制，允许我们检索输出字节，例如内存缓冲区或写入的文件。另一方面，如果没有可用的输出字节副本，我们可能会考虑使用DrainablReOutputStr之类的技术在写入字节时获取字节的副本。

## 5. 总结

总之，在编程中，有些场景将OutputStrеam转换为Java中的字节数组可能是一种有用的操作。我们了解了如何使用Apache Commons IO库中的FileUtils.readFileToByteArray()读取由FilеOutputStrеam生成的文件，以及如何使用自定义DrainablеOutputStrеam获取给定OutputStrеam的写入字节副本的更通用方法。