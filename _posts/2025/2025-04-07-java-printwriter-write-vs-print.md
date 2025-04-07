---
layout: post
title:  Java中的PrintWriter write()与print()方法
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在本文中，我们将讨论[Java IO包](https://www.baeldung.com/java-io)中的PrintWriter类。具体来说，我们将讨论它的两个方法write()和print()以及它们的区别。

PrintWriter类将对象的格式化表示打印到文本输出流，此类中的方法不会抛出[I/O异常](https://www.baeldung.com/java-exceptions)。但是，它的某些构造函数可能会抛出异常。要使用这些方法，我们必须调用PrintWriter构造函数并提供文件、文件名或输出流作为参数。

## 2. PrintWriter.write()

**write()方法有5个重载版本，两个用于char，两个用于String，一个用于int**。此方法只是我们可以写入控制台或[文件](https://www.baeldung.com/java-write-to-file)的可能方法之一。

此外，char和String版本可以写入整个char[]或String，也可以写入数组或String的部分内容。此外，int版本写入单个字符-相当于给定十进制输入的ASCII符号。

首先，让我们看一下write(int c)版本：

```java
@Test
void whenUsingWriteInt_thenASCIICharacterIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    printWriter.write(48);
    printWriter.close();

    assertEquals(0, outputFromPrintWriter());
}
```

我们将48作为参数传递给write()方法，并获得了0作为输出。此外，如果我们检查ASCII表，我们会看到48 DEC输入对应的符号是数字0。如果我们传递另一个值，比如说64，那么打印的输出将是数字4。

这里，outputFromPrintWriter()只是一个服务方法，它读取write()方法在文件中写入的内容，因此我们可以比较这些值：

```java
Object outputFromPrintWriter;

Object outputFromPrintWriter() {
    try (BufferedReader br = new BufferedReader(new FileReader("output.txt"))){
        outputFromPrintWriter = br.readLine();
    } catch (IOException e){
        e.printStackTrace();
        Assertions.fail();
    }
    return outputFromPrintWriter;
}
```

现在，让我们看一下第二个版本-write(char[] buf，int off，int len)，它从给定的起始位置到给定的长度写入数组的一部分：

```java
@Test
void whenUsingWriteCharArrayFromOffset_thenCharArrayIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    printWriter.write(new char[]{'A','/','&','4','E'}, 1, 4);
    printWriter.close();

    assertEquals("/&4E", outputFromPrintWriter());
}
```

从上面的测试中我们可以看到，write()方法从所需偏移量中获取我们指定的4个字符，并将它们打印到output.txt文件中。

让我们分析一下write(String s, int off, int len)方法的第二个版本，该方法从给定的起始位置开始，写入字符串的一部分，直到给定的长度：

```java
@Test
void whenUsingWriteStringFromOffset_thenLengthOfStringIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    printWriter.write("StringExample", 6, 7 );
    printWriter.close();

    assertEquals("Example", outputFromPrintWriter());
}
```

## 3. PrintWriter.print()

**print()方法本身有9个重载版本**，它可以接收以下类型作为参数：boolean、char、char[]、double、float、int、long、Object和String。

**print()方法的行为在其变体中是相似的，对于除String和char之外的所有类型**，String.valueOf()方法生成的字符串都会被转换为字节。此转换是根据平台的默认字符编码进行的，并且字节的写入方式与write(int)版本相同。

首先，让我们看一下print(boolean b)版本：

```java
@Test
void whenUsingPrintBoolean_thenStringValueIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    printWriter.print(true);
    printWriter.close();

    assertEquals("true", outputFromPrintWriter());
}
```

现在，让我们看一下print(char c)：

```java
@Test
void whenUsingPrintChar_thenCharIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    printWriter.print('A');
    printWriter.close();

    assertEquals("A", outputFromPrintWriter());
}
```

print(int i)的工作原理如下：

```java
@Test
void whenUsingPrintInt_thenValueOfIntIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    printWriter.print(420);
    printWriter.close();

    assertEquals("420", outputFromPrintWriter());
}
```

让我们看看print(String s)版本的输出：

```java
@Test
void whenUsingPrintString_thenStringIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    printWriter.print("RandomString");
    printWriter.close();

    assertEquals("RandomString", outputFromPrintWriter());
}
```

以下是对print(Object obj)的测试：

```java
@Test
void whenUsingPrintObject_thenObjectToStringIsPrinted() throws FileNotFoundException {
    PrintWriter printWriter = new PrintWriter("output.txt");

    Map example = new HashMap();

    printWriter.print(example);
    printWriter.close();

    assertEquals(example.toString(), outputFromPrintWriter());
}
```

从上面的例子我们可以看出，**调用print()方法并传递一个对象作为参数会打印该对象的toString()表示形式**。

## 4. write()和print()方法之间的区别

这两种方法之间的差异很细微，所以我们需要注意。

**write(int)只写入一个字符，它输出与传递的参数等效的ASCII符号**。

**此外，print(typeOfData)通过调用String.valueOf(typeOfData)将char、int等类型的参数转换为字符串**，该字符串根据平台默认的字符编码转换为字节，其字节的写入方式与write(int)相同。

与print()方法不同，write()方法仅处理单个字符、字符串和字符数组。print()方法涵盖许多参数类型，使用String.valueOf()将它们转换为可打印的字符串。

## 5. 总结

在本教程中，我们探索了PrintWriter类，更具体地说，是write()和print()方法。如我们所见，PrintWriter类提供了多种方法来帮助我们将数据打印到输出，write()方法打印传递给它的字符，而print()方法转换输出。