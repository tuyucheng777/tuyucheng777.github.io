---
layout: post
title:  列出可用的加密算法
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在这个快速教程中，我们将介绍Java中的[Cipher](https://www.baeldung.com/java-cipher-class)类。然后，我们将看到如何列出可用的密码算法及其提供程序。

## 2. Cipher类

位于javax.crypto包中的Cipher类是Java密码学扩展(JCE)框架的核心。该框架提供了一组用于数据加密、解密和哈希的加密算法。

## 3. 列出密码算法

**我们可以通过以请求的转换名称作为参数调用Cipher.getInstance()静态方法来实例化Cipher对象**：

```java
Cipher cipher = Cipher.getInstance("AES");
```

在某些情况下，我们需要获取可用密码算法及其提供程序的列表。例如，我们想根据类路径中存在的库检查特定算法是否可用。

首先，**我们需要使用[Security.getProviders()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/Security.html#getProviders())方法获取已注册提供程序的列表。然后，在Provider对象上调用[getServices()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/Provider.html#getServices())方法将返回此Provider支持的所有服务的不可修改集合**：

```java
for (Provider provider : Security.getProviders()) {
    for (Provider.Service service : provider.getServices()) {
        String algorithm = service.getAlgorithm();
        // ...
    }
}
```

可用算法列表：

```text
SHA3-224
NONEwithDSA
DSA
JavaLoginConfig
DSA
SHA3-384
SHA3-256
SHA1withDSA
...
```

但是，**并非所有列出的算法都支持作为[Cipher.getInstance()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html#getInstance(java.lang.String))静态方法的转换**。例如，使用哈希算法SHA3-224实例化Cipher对象将抛出NoSuchAlgorithmException：

```java
Cipher cipher = Cipher.getInstance("SHA3-224");
```

让我们看一下运行时异常消息：

```text
java.security.NoSuchAlgorithmException: Cannot find any provider supporting SHA3-224
```

**因此，我们需要过滤列表并保留Cipher类型的服务**。我们可以使用Java Stream来过滤和收集兼容算法的名称列表：

```java
List<String> algorithms = Arrays.stream(Security.getProviders())
    .flatMap(provider -> provider.getServices().stream())
    .filter(service -> "Cipher".equals(service.getType()))
    .map(Provider.Service::getAlgorithm)
    .collect(Collectors.toList());
    // ...
```

结果将类似于：

```text
AES_192/CBC/NoPadding
AES_192/OFB/NoPadding
AES_192/CFB/NoPadding
AESWrap_192
PBEWithHmacSHA224AndAES_256
AES_192/ECB/NoPadding
AES_192/GCM/NoPadding
ChaCha20-Poly1305
PBEWithHmacSHA384AndAES_128
AES_128/ECB/NoPadding
AES_128/OFB/NoPadding
AES_128/CBC/NoPadding
...
```

## 4. 总结

在本教程中，我们首先了解了Cipher类。然后，我们学习了如何列出可用的加密算法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-algorithms)上获得。