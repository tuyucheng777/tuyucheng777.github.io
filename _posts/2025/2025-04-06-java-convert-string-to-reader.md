---
layout: post
title:  String转为Reader
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个快速教程中，我们将了解如何将String转换为Reader，首先使用纯Java，然后使用Guava，最后使用Commons IO库。

## 2. 使用纯Java

让我们从Java解决方案开始：

```java
@Test
public void givenUsingPlainJava_whenConvertingStringIntoReader_thenCorrect() throws IOException {
    String initialString = "With Plain Java";
    Reader targetReader = new StringReader(initialString);
    targetReader.close();
}
```

如你所见，StringReader开箱即用，可用于这种简单的转换。

## 3. Guava

接下来是Guava解决方案：

```java
@Test
public void givenUsingGuava_whenConvertingStringIntoReader_thenCorrect() throws IOException {
    String initialString = "With Google Guava";
    Reader targetReader = CharSource.wrap(initialString).openStream();
    targetReader.close();
}
```

我们在这里使用通用的CharSource抽象，允许我们从中打开一个Reader。

## 4. 使用Apache Commons IO

最后，这是Commons IO解决方案，也使用现成的Reader实现：

```java
@Test
public void givenUsingCommonsIO_whenConvertingStringIntoReader_thenCorrect() throws IOException {
    String initialString = "With Apache Commons IO";
    Reader targetReader = new CharSequenceReader(initialString);
    targetReader.close();
}
```

以上就是将Java String转换为Reader的3种快速方法。