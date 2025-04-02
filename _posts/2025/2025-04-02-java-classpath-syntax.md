---
layout: post
title:  Linux与Windows中的Java类路径语法
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

[类路径](https://en.wikipedia.org/wiki/Classpath)是Java世界中的一个基本概念，当我们编译或启动Java应用程序时，JVM会在类路径中查找并加载类。

我们可以通过java/javac命令的-cp选项或通过CLASSPATH环境变量来定义类路径中的元素，无论我们采用哪种方法来设置类路径，我们都需要遵循类路径语法。

在本快速教程中，我们将讨论类路径语法，尤其是Windows和Linux操作系统上的类路径分隔符。

## 2. 类路径分隔符

类路径语法实际上非常简单：由路径分隔符分隔的路径列表。但是，[路径分隔符](https://www.baeldung.com/java-file-vs-file-path-separator#path-separator)本身是系统相关的。

**在Microsoft Windows系统上使用分号(;)用作分隔符，而在类Unix系统上使用冒号(:)**：

```shell
# On Windows system:
CLASSPATH="PATH1;PATH2;PATH3"

# On Linux system:
CLASSPATH="PATH1:PATH2:PATH3"
```

## 3. Linux上误导性的手册页

我们了解到类路径分隔符可能因操作系统而异。

但是，如果我们仔细查看Linux上的Java手册页，它说类路径分隔符是分号(;)。

例如，最新(17)OpenJDK的java命令的手册页显示：

>   –class-path classpath、-classpath classpath或-cp classpath以分号(;)分隔的目录列表、JAR存档和ZIP存档以搜索类文件。

另外，我们可以在[Oracle JDK](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html#standard-options-for-java)的手册中找到确切的文本。

这是因为Java目前针对不同的系统使用相同的手册内容，今年早些时候已经创建了相应的[错误问题](https://bugs.openjdk.java.net/browse/JDK-8262004)。

此外，Java已明确记录路径分隔符依赖于File类的[pathSeparatorChar](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/File.html#pathSeparatorChar)字段的系统。

## 4. 总结

在这篇简短的文章中，我们讨论了不同操作系统上的类路径语法。

此外，我们还讨论了有关Linux上Java手册页中的路径分隔符的一个错误。

我们应该记住，路径分隔符是系统相关的。在类Unix系统上使用冒号，而在Microsoft Windows系统上使用分号。