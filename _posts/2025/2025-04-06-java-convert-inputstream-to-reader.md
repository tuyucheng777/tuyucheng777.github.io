---
layout: post
title:  InputStream转为Reader
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1, 概述

在本快速教程中，我们将研究如何使用Java、Guava、Apache Commons IO将InputStream转换为Reader。

## 2. 普通Java

首先，让我们看一下简单的Java解决方案-使用现成的InputStreamReader：

```java
@Test
public void givenUsingPlainJava_whenConvertingInputStreamIntoReader_thenCorrect() throws IOException {
    InputStream initialStream = new ByteArrayInputStream("With Java".getBytes());
    
    Reader targetReader = new InputStreamReader(initialStream);

    targetReader.close();
}
```

## 3. Guava

接下来让我们看看Guava解决方案，使用中间字节数组和字符串：

```java
@Test
public void givenUsingGuava_whenConvertingInputStreamIntoReader_thenCorrect() throws IOException {
    InputStream initialStream = ByteSource.wrap("With Guava".getBytes()).openStream();
    
    byte[] buffer = ByteStreams.toByteArray(initialStream);
    Reader targetReader = CharSource.wrap(new String(buffer)).openStream();

    targetReader.close();
}
```

请注意，Java解决方案比这种方法更简单。

## 4. 使用Commons IO

最后使用Apache Commons IO的解决方案-也使用中间字符串：

```java
@Test
public void givenUsingCommonsIO_whenConvertingInputStreamIntoReader_thenCorrect() 
  throws IOException {
    InputStream initialStream = IOUtils.toInputStream("With Commons IO");
    
    byte[] buffer = IOUtils.toByteArray(initialStream);
    Reader targetReader = new CharSequenceReader(new String(buffer));

    targetReader.close();
}
```

以上就是将输入流转换为Java Reader的3种快速方法。