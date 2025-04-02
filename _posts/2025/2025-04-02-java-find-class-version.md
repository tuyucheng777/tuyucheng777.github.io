---
layout: post
title:  查找Java .class版本的指南
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本教程中，我们将了解如何查找.class文件的Java发布版本。此外，我们还将了解如何检查jar文件中的Java版本。

## 2. Java中的.class版本

当Java文件被编译时，会生成一个.class文件。在某些情况下，我们需要找到编译后的类文件的Java发布版本。**每个Java主版本为其生成的.class文件分配一个主版本**。

在此表中，我们将.class的主版本号映射到引入该类版本的JDK版本，并显示该版本号的十六进制表示形式：

|Java发布  | class主要版本 | 十六进制 |
| :--------: |:---------:| :------: |
|JavaSE 18 |    62     |   003e   |
|JavaSE 17 |    61     |   003d   |
|JavaSE 16 |    60     |   003c   |
|JavaSE 15 |    59     |   003b   |
|JavaSE 14 |    58     |   003a   |
|JavaSE 13 |    57     |   0039   |
|JavaSE 12 |    56     |   0038   |
|JavaSE 11 |    55     |   0037   |
|JavaSE 10 |    54     |   0036   |
|JavaSE 9  |    53     |   0035   |
|JavaSE 8  |    52     |   0034   |
|JavaSE 7  |    51     |   0033   |
|JavaSE 6  |    50     |   0032   |
|JavaSE 5  |    49     |   0031   |
|  JDK 1.4   |    48     |   0030   |
|  JDK 1.3   |    47     |   002f   |
|  JDK 1.2   |    46     |   002e   |
|  JDK 1.1   |    45     |   002d   |

## 3. .class版本的javap命令

让我们创建一个简单的类并使用JDK 8构建它：

```java
public class Sample {
    public static void main(String[] args) {
        System.out.println("Baeldung tutorials");
    }
}
```

**为了识别类文件的版本，我们可以使用Java类文件反汇编器[javap](https://www.baeldung.com/java-class-view-bytecode#javaCommandLine)**。

下面是javap命令的语法：

```shell
javap [option] [classname]
```

让我们以Sample.class为例：

```shell
javap -verbose Sample

//stripped output ..
..
..
Compiled from "Sample.java"
public class test.Sample
  minor version: 0
  major version: 52
..
..
```

在javap命令的输出中我们可以看到，主版本是52，说明它是针对JDK 8的。

虽然javap提供了很多细节，但我们只关心主版本。

对于任何基于Linux的系统，我们可以使用以下命令仅获取主版本：

```shell
javap -verbose Sample | grep major
```

同样，对于Windows系统，我们可以使用以下命令：

```shell
javap -verbose Sample | findstr major
```

在我们的示例中，这给出了主版本52。

**需要注意的是，此版本值并不表示该应用程序是使用相应的JDK构建的，类文件版本可以与用于编译的JDK不同**。

**例如，如果我们使用JDK 11构建代码，它应该生成版本为55的.class文件。但是，如果我们在编译期间传递[-target](https://www.baeldung.com/java-source-target-options) 8，则.class文件的版本将为52**。

## 4. .class版本的hexdump

也可以使用任何十六进制编辑器检查版本，Java类文件遵循[规范](https://en.wikipedia.org/wiki/Java_class_file)。我们看一下它的结构：

```text
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    // other details
}
```

在这里，类型u1、u2和u4分别表示无符号的一、二和四字节整数。u4是标识类文件格式的魔数，它的值为0xCAFEBABE，而u2是主版本。

**对于基于Linux的系统，我们可以使用[hexdump](https://www.baeldung.com/linux/create-hex-dump#using-hexdump)实用程序来解析任何.class文件**：

```shell
> hexdump -v Sample.class
0000000 ca fe ba be 00 00 00 34 00 22 07 00 02 01 00 0b
0000010 74 65 73 74 2f 53 61 6d 70 6c 65 07 00 04 01 00
...truncated
```

在这个例子中，我们使用JDK 8编译。第一行的7和8索引提供了类文件的主要版本，因此，0034是十六进制表示，JDK 8是对应的版本号(从我们前面看到的映射表)。

另外，**我们可以使用hexdump直接获取十进制形式的主发布版本号**：

```shell
> hexdump -s 7 -n 1 -e '"%d"' Sample.class
52
```

这里，输出52是对应JDK 8的类版本。

## 5. jar版本

Java生态系统中的jar文件由一组捆绑在一起的类文件组成，为了找出jar是哪个Java版本构建或编译的，**我们可以提取jar文件并使用[javap](https://www.baeldung.com/java-class-view-bytecode#javaCommandLine)或[hexdump](https://www.baeldung.com/linux/create-hex-dump#using-hexdump)检查.class文件版本**。

jar文件中还有一个[MANIFEST.MF](https://www.baeldung.com/java-jar-manifest)文件，里面包含了一些使用的JDK的头信息。

例如，Build-Jdk或Created-By标头根据jar的构建方式存储JDK值：

```text
Build-Jdk: 17.0.4
```

或者

```text
Created-By: 17.0.4
```

## 6. 总结

在本文中，我们学习了如何查找.class文件的Java版本，我们了解了javap和hexdump命令及其查找版本的用法。此外，我们还了解了如何检查jar文件的Java版本。