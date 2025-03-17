---
layout: post
title:  在Java中将UUID存储为Base64字符串
category: java
copyright: java
excerpt: Java UUID
---

## 1. 概述

使用Base64编码字符串是存储通用唯一标识符(UUID)的一种广泛采用的方法，与标准UUID字符串表示相比，这提供了更紧凑的结果。**在本文中，我们将探讨将UUID编码为Base64字符串的各种方法**。

## 2. 使用byte[]和Base64.Encoder进行编码

我们将从使用byte[]和Base64.Encoder的最直接的编码方法开始。

### 2.1 编码

我们将从UUID位创建一个字节数组，**为此，我们将从UUID中取出最高有效位和最低有效位，并将它们分别放在数组中的位置0-7和8-15**：

```java
byte[] convertToByteArray(UUID uuid) {
    byte[] result = new byte[16];

    long mostSignificantBits = uuid.getMostSignificantBits();
    fillByteArray(0, 8, result, mostSignificantBits);

    long leastSignificantBits = uuid.getLeastSignificantBits();
    fillByteArray(8, 16, result, leastSignificantBits);

    return result;
}
```

在填充方法中，我们将位移动到数组中，将它们转换为字节，并在每次迭代中移动8位：

```java
void fillByteArray(int start, int end, byte[] result, long bits) {
    for (int i = start; i < end; i++) {
        int shift = i * 8;
        result[i] = (byte) ((int) (255L & bits >> shift));
    }
}
```

下一步，我们将使用JDK中的[Base64.Encoder](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Base64.Encoder.html)将字节数组编码为字符串：

```java
UUID originalUUID = UUID.fromString("cc5f93f7-8cf1-4a51-83c6-e740313a0c6c");

@Test
void givenEncodedString_whenDecodingUsingBase64Decoder_thenGiveExpectedUUID() {
    String expectedEncodedString = "UUrxjPeTX8xsDDoxQOfGgw==";
    byte[] uuidBytes = convertToByteArray(originalUUID);
    String encodedUUID = Base64.getEncoder().encodeToString(uuidBytes);
    assertEquals(expectedEncodedString, encodedUUID);
}
```

可以看到，得到的值正是我们预期的。

### 2.2 解码

要从Base64编码的字符串解码UUID，我们可以按照以下方式执行相反的操作：

```java
@Test
public void givenEncodedString_whenDecodingUsingBase64Decoder_thenGiveExpectedUUID() {
    String expectedEncodedString = "UUrxjPeTX8xsDDoxQOfGgw==";
    byte[] decodedBytes = Base64.getDecoder().decode(expectedEncodedString);
    UUID uuid = convertToUUID(decodedBytes);
}
```

首先，我们使用[Base64.Decoder](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Base64.Decoder.html)从编码字符串中获取一个字节数组，并调用转换方法从该数组中创建UUID：

```java
UUID convertToUUID(byte[] src) {
    long mostSignificantBits = convertBytesToLong(src, 0);
    long leastSignificantBits = convertBytesToLong(src, 8);

    return new UUID(mostSignificantBits, leastSignificantBits);
}
```

我们将数组的各个部分转换为最高和最低有效位长表示，并使用它们创建UUID。

转换方法如下：

```java
long convertBytesToLong(byte[] uuidBytes, int start) {
    long result = 0;

    for(int i = 0; i < 8; i++) {
        int shift = i * 8;
        long bits = (255L & (long)uuidBytes[i + start]) << shift;
        long mask = 255L << shift;
        result = result & ~mask | bits;
    }

    return result;
}
```

在这个方法中，我们遍历字节数组，将每个字节转换为位，然后将它们移动到我们的结果中。

**我们可以看到，解码的最终结果将与我们用于编码的原始UUID相匹配**。

## 3. 使用ByteBuffer和Base64.getUrlEncoder()进行编码

使用JDK的标准功能，我们可以简化上面写的代码。

### 3.1 编码

使用[ByteBuffer](https://www.baeldung.com/java-bytebuffer)，我们只需几行代码就可以将UUID转换为字节数组：

```java
ByteBuffer byteBuffer = ByteBuffer.wrap(new byte[16]);
byteBuffer.putLong(originalUUID.getMostSignificantBits());
byteBuffer.putLong(originalUUID.getLeastSignificantBits());
```

我们创建了一个包装字节数组的缓冲区，并放入了UUID中的最高和最低有效位。

为了编码目的，我们这次将使用[Base64.getUrlEncoder()](https://docs.oracle.com/en/java/javase/21/docs//api/java.base/java/util/Base64.html#getUrlEncoder())：

```java
String encodedUUID = Base64.getUrlEncoder().encodeToString(byteBuffer.array());
```

因此，我们用4行代码创建了一个Base64编码的UUID：

```java
@Test
public void givenUUID_whenEncodingUsingByteBufferAndBase64UrlEncoder_thenGiveExpectedEncodedString() {
    String expectedEncodedString = "zF-T94zxSlGDxudAMToMbA==";
    ByteBuffer byteBuffer = ByteBuffer.wrap(new byte[16]);
    byteBuffer.putLong(originalUUID.getMostSignificantBits());
    byteBuffer.putLong(originalUUID.getLeastSignificantBits());
    String encodedUUID = Base64.getUrlEncoder().encodeToString(byteBuffer.array());
    assertEquals(expectedEncodedString, encodedUUID);
}
```

### 3.2 解码

我们可以使用ByteBuffer和[Base64.UrlDecoder()](https://docs.oracle.com/en/java/javase/21/docs//api/java.base/java/util/Base64.html#getUrlDecoder())执行相反的操作：

```java
@Test
void givenEncodedString_whenDecodingUsingByteBufferAndBase64UrlDecoder_thenGiveExpectedUUID() {
    String expectedEncodedString = "zF-T94zxSlGDxudAMToMbA==";
    byte[] decodedBytes = Base64.getUrlDecoder().decode(expectedEncodedString);
    ByteBuffer byteBuffer = ByteBuffer.wrap(decodedBytes);
    long mostSignificantBits = byteBuffer.getLong();
    long leastSignificantBits = byteBuffer.getLong();
    UUID uuid = new UUID(mostSignificantBits, leastSignificantBits);
    assertEquals(originalUUID, uuid);
}
```

我们可以看到，我们成功地从编码字符串中解码出了预期的UUID。

## 4. 减少编码UUID的长度

正如我们在前面几节中看到的，Base64默认在末尾包含“==”。为了节省更多字节，我们可以修剪这个结尾。

为此，我们可以将编码器配置为不添加填充：

```java
String encodedUUID = Base64.getUrlEncoder().withoutPadding().encodeToString(byteBuffer.array());

assertEquals(expectedEncodedString, encodedUUID);
```

结果，我们可以看到没有多余字符的编码字符串。**无需更改我们的解码器，因为它将以相同的方式处理编码字符串的两个变体**。

## 5. 使用Apache Commons的转换实用程序和编解码器实用程序进行编码

在本节中，我们将使用[Apache Commons Conversion](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/Conversion.html)实用程序中的uuidToByteArray来创建UUID字节数组。此外，我们还将使用[Apache Commons Base64](https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/binary/Base64.html)实用程序中的encodeBase64URLSafeString。

### 5.1 依赖

为了演示这种编码方法，我们将使用Apache Commons Lang库，让我们将其[依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

我们将使用的另一个[依赖](https://mvnrepository.com/artifact/commons-codec/commons-codec)是commons-codec：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.16.0</version>
</dependency>
```

### 5.2 编码

我们只需两行代码就可以对UUID进行编码：

```java
@Test
void givenUUID_whenEncodingUsingApacheUtils_thenGiveExpectedEncodedString() {
    String expectedEncodedString = "UUrxjPeTX8xsDDoxQOfGgw";
    byte[] bytes = Conversion.uuidToByteArray(originalUUID, new byte[16], 0, 16);
    String encodedUUID = encodeBase64URLSafeString(bytes);
    assertEquals(expectedEncodedString, encodedUUID);
}
```

可以看到，结果已经被修剪并且不包含待定的结尾。

### 5.3 解码

我们将从Apache Commons调用[Base64.decodeBase64()](https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/binary/Base64.html#decodeBase64-java.lang.String-)和[Conversion.byteArrayToUuid()](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/Conversion.html#byteArrayToUuid-byte:A-int-)进行反向操作：

```java
@Test
void givenEncodedString_whenDecodingUsingApacheUtils_thenGiveExpectedUUID() {
    String expectedEncodedString = "UUrxjPeTX8xsDDoxQOfGgw";
    byte[] decodedBytes = decodeBase64(expectedEncodedString);
    UUID uuid = Conversion.byteArrayToUuid(decodedBytes, 0);
    assertEquals(originalUUID, uuid);
}
```

我们成功获取了原始的UUID。

## 6. 总结

UUID是一种广泛使用的数据类型，对其进行编码的方法之一是使用Base64。在本文中，我们探讨了几种将UUID编码为Base64的方法。