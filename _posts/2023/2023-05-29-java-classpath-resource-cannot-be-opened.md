---
layout: post
title:  如何在加载资源时避免Java FileNotFoundException
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将探讨在Java应用程序中读取资源文件时可能出现的问题：在运行时，资源文件夹很少与我们的源代码位于磁盘上的同一位置。

让我们看看Java如何让我们在代码打包后访问资源文件。

## 2. 读取文件

假设我们的应用程序在启动期间读取一个文件：

```java
try (FileReader fileReader = new FileReader("src/main/resources/input.txt"); 
     BufferedReader reader = new BufferedReader(fileReader)) {
    String contents = reader.lines().collect(Collectors.joining(System.lineSeparator()));
}
```

如果我们在IDE中运行上述代码，文件加载时不会出现错误。这是因为我们的IDE使用我们的项目目录作为其当前工作目录，而src/main/resources目录就在那里供应用程序读取。

现在假设我们使用[Maven JAR插件](https://www.baeldung.com/executable-jar-with-maven)将代码打包为JAR。

当我们在命令行运行它时：

```shell
java -jar core-java-io2.jar
```

我们会看到以下错误：

```plaintext
Exception in thread "main" java.io.FileNotFoundException: 
    src/main/resources/input.txt (No such file or directory)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at java.io.FileInputStream.<init>(FileInputStream.java:93)
	at java.io.FileReader.<init>(FileReader.java:58)
	at cn.tuyucheng.taketoday.resource.MyResourceLoader.loadResourceWithReader(MyResourceLoader.java:14)
	at cn.tuyucheng.taketoday.resource.MyResourceLoader.main(MyResourceLoader.java:37)
```

## 3. 源代码与编译代码

当我们构建JAR时，资源被放置在打包工件的根目录中。

在我们的示例中，我们看到源代码设置在源代码目录的src/main/resources中有input.txt。

然而，在相应的JAR结构中，我们看到：

```plaintext
META-INF/MANIFEST.MF
META-INF/
com/
com/baeldung/
com/baeldung/resource/
META-INF/maven/
META-INF/maven/cn.tuyucheng.taketoday/
META-INF/maven/cn.tuyucheng.taketoday/core-java-io-files/
input.txt
com/baeldung/resource/MyResourceLoader.class
META-INF/maven/cn.tuyucheng.taketoday/core-java-io-files/pom.xml
META-INF/maven/cn.tuyucheng.taketoday/core-java-io-files/pom.properties
```

此处，input.txt位于JAR的根目录中。因此，当代码执行时，我们将看到FileNotFoundException。

即使我们将路径更改为/input.txt，原始代码也无法加载此文件，因为**资源通常无法作为磁盘上的文件进行寻址**。资源文件打包在JAR中，因此我们需要一种不同的方式来访问它们。

## 4. Resources

让我们改为使用资源加载从类路径而不是特定文件位置加载资源。无论代码如何打包，这都将起作用：

```java
try (InputStream inputStream = getClass().getResourceAsStream("/input.txt");
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))) {
    String contents = reader.lines().collect(Collectors.joining(System.lineSeparator()));
}
```

ClassLoader.getResourceAsStream()查看给定资源的类路径。getResourceAsStream()输入中的前导斜杠告诉加载器从类路径的基址读取。我们的JAR文件的内容在classpath中，所以这个方法有效。

**IDE通常在其类路径中包含src/main/resources，从而找到这些文件**。

## 5. 总结

在这篇简短的文章中，我们将加载文件作为类路径资源来实现，以使我们的代码无论以何种方式打包都能够始终如一地工作。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-1)上获得。
