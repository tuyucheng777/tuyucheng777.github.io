---
layout: post
title:  Java中字符串SHA-1摘要的十六进制表示
category: algorithms
copyright: algorithms
excerpt: 字符串
---

## 1. 概述

众所周知，确保数据完整性和安全性在软件工程中至关重要。**一个简单的方法是使用[加密哈希函数](https://www.baeldung.com/cs/md5-vs-sha-algorithms)，例如安全哈希算法1(SHA-1)，它广泛用于[校验和](https://www.baeldung.com/java-checksums)以及数据完整性验证**。

在本教程中，我们将探讨如何使用三种方法在Java中生成字符串的SHA-1哈希的十六进制(hex)表示形式。

## 2. 示例设置

在本教程中，我们将生成示例输入的SHA-1摘要的十六进制表示形式，并将其与预期的十六进制输出进行比较：

```java
String input = "Hello, World";
String expectedHexValue= "907d14fb3af2b0d4f18c2d46abe8aedce17367bd";
```

## 3. 使用MessageDigest

MessageDigest是一个内置的Java类，它是java.security包的一部分，提供了一种生成字符串SHA-1摘要的简便方法。

下面是一些使用MessageDigest生成SHA-1摘要的十六进制表示的示例代码：

```java
@Test
void givenMessageDigest_whenUpdatingWithData_thenDigestShouldMatchExpectedValue() throws NoSuchAlgorithmException {
    MessageDigest md = MessageDigest.getInstance("SHA-1");
    md.update(input.getBytes(StandardCharsets.UTF_8));
    byte[] digest = md.digest();
        
    StringBuilder hexString = new StringBuilder();
        
    for (byte b : digest) {
        hexString.append(String.format("%02x", b));
    }

    assertEquals(expectedHexValue, hexString.toString());
}
```

首先，我们创建一个MessageDigest的新实例，并使用SHA-1算法对其进行初始化。然后，我们调用MessageDigest的update()方法，并将输入转换为byte类型。接下来，我们创建一个byte[]类型的新变量digest，此变量保存添加到MessageDigest对象的数据的加密哈希值。

此外，我们循环遍历摘要数组中的字节，并将每个字节追加到十六进制字符串，String.format()方法有助于将字节值格式化为十六进制字符串。

最后，我们断言输入等于预期的十六进制值。

## 4. 使用Apache Commons Codec

**Apache Commons Codec库提供了一个名为DigestUtils的类，它简化了生成SHA-1摘要的十六进制表示的过程**。

要使用这个库，我们需要将它的[依赖](https://mvnrepository.com/artifact/commons-codec/commons-codec)添加到pom.xml中：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

接下来我们看看如何使用DigestUtils类：

```java
@Test
void givenDigestUtils_whenCalculatingSHA1Hex_thenDigestShouldMatchExpectedValue() {
    assertEquals(expectedHexValue, DigestUtils.sha1Hex(input));
}
```

这里，我们调用DigestUtils类的sha1Hex()方法。该方法用于计算字符串的SHA-1哈希值，并以十六进制形式返回结果。此外，我们检查输入是否等于预期的十六进制值。

## 5. 使用Google Guava

**[Guava](https://www.baeldung.com/guava-guide)是Google开发的一个库，它提供了一个Hashing类，可用于生成SHA-1摘要的十六进制表示形式**。

为了使用这个库，我们需要将它的[依赖](https://mvnrepository.com/artifact/com.google.guava/guava)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
</dependency>
```

让我们看看如何使用Guava库：

```java
@Test
void givenHashingLibrary_whenCalculatingSHA1Hash_thenDigestShouldMatchExpectedValue() {
    assertEquals(expectedHexValue, Hashing.sha1().hashString(input, StandardCharsets.UTF_8).toString());
}
```

在上面的示例代码中，我们调用Hashing类的sha1()方法，使用UTF-8字符编码计算SHA-1哈希值。然后，我们断言输出等于预期结果。

## 6. 总结

在本文中，我们学习了三种在Java中生成字符串SHA-1摘要十六进制表示的方法。内置的MessageDigest类不需要额外的依赖，此外，Apache Commons Codec库和Guava库的使用也更加方便。