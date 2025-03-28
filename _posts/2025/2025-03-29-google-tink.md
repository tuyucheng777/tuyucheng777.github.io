---
layout: post
title:  Google Tink指南
category: libraries
copyright: libraries
excerpt: Google Tink
---

## 1. 简介

如今，许多开发人员使用加密技术来保护用户数据。

在密码学中，微小的实现错误可能会产生严重的后果，而了解如何正确实现密码学是一项复杂且耗时的任务。

**在本教程中，我们将介绍[Tink](https://github.com/google/tink)-一个多语言、跨平台的加密库，它可以帮助我们实现安全的加密代码**。

## 2. 依赖

我们可以使用Maven或Gradle来导入Tink。

对于我们的教程，我们只需添加[Tink的Maven依赖](https://mvnrepository.com/artifact/com.google.crypto.tink/tink)：

```xml
<dependency>
    <groupId>com.google.crypto.tink</groupId>
    <artifactId>tink</artifactId>
    <version>1.2.2</version>
</dependency>
```

虽然我们也可以使用Gradle：

```groovy
dependencies {
    compile 'com.google.crypto.tink:tink:latest'
}
```

## 3. 初始化

**在使用任何Tink API之前，我们都需要初始化它们**。

如果我们需要使用Tink中所有原语的所有实现，可以使用TinkConfig.register()方法：

```java
TinkConfig.register();
```

例如，如果我们只需要AEAD原语，我们可以使用AeadConfig.register()方法：

```java
AeadConfig.register();
```

还为每个实现提供了可定制的初始化。

## 4. Tink原语

**该库使用的主要对象称为原语，根据类型的不同，包含不同的加密功能**。

一个原语可以有多个实现：

|    原语    |                                实现                                |
|:--------:|:----------------------------------------------------------------:|
|   AEAD   |   AES-EAX、AES-GCM、AES-CTR-HMAC、KMS Envelope、CHACHA20-POLY1305    |
|  流式AEAD  |          AES-GCM-HKDF-STREAMING、AES-CTR-HMAC-STREAMING           |
| 确定性 AEAD |                           AEAD：AES-SIV                           |
|   MAC    |                            HMAC-SHA2                             |
|   数字签名   |                      NIST曲线上的ECDSA，ED25519                       |
|   混合加密   |                 ECIES与AEAD和HKDF，(NaCl CryptoBox)                 |

我们可以通过调用相应工厂类的getPrimitive()方法并向其传递一个KeysetHandle来获取原语：

```java
Aead aead = AeadFactory.getPrimitive(keysetHandle);
```

### 4.1 KeysetHandle

**为了提供加密功能，每个原语都需要一个包含所有密钥材料和参数的密钥结构**。

Tink提供了一个对象KeysetHandle，它用一些附加参数和元数据包装了一个键集。

因此，在实例化原语之前，我们需要创建一个KeysetHandle对象：

```java
KeysetHandle keysetHandle = KeysetHandle.generateNew(AeadKeyTemplates.AES256_GCM);
```

生成密钥后，我们可能想要将其保留下来：

```java
String keysetFilename = "keyset.json";
CleartextKeysetHandle.write(keysetHandle, JsonKeysetWriter.withFile(new File(keysetFilename)));
```

然后，我们就可以加载它：

```java
String keysetFilename = "keyset.json";
KeysetHandle keysetHandle = CleartextKeysetHandle.read(JsonKeysetReader.withFile(new File(keysetFilename)));
```

## 5. 加密

Tink提供了多种应用AEAD算法的方法，我们来看看。

### 5.1 AEAD

AEAD提供带有关联数据的经过身份验证的加密，这意味着**我们可以加密纯文本，并且可以选择提供需要身份验证但无需加密的关联数据**。

请注意，该算法确保相关数据的真实性和完整性，但不确保其保密性。

正如我们之前看到的，要使用AEAD实现之一加密数据，我们需要初始化库并创建一个keysetHandle：

```java
AeadConfig.register();
KeysetHandle keysetHandle = KeysetHandle.generateNew(AeadKeyTemplates.AES256_GCM);
```

一旦完成了这些，我们就可以获取原语并加密所需的数据：

```java
String plaintext = "tuyucheng";
String associatedData = "Tink";

Aead aead = AeadFactory.getPrimitive(keysetHandle); 
byte[] ciphertext = aead.encrypt(plaintext.getBytes(), associatedData.getBytes());
```

接下来，我们可以使用decrypt()方法解密密文：

```java
String decrypted = new String(aead.decrypt(ciphertext, associatedData.getBytes()));
```

### 5.2 流式AEAD

类似地，**当要加密的数据太大而无法通过一个步骤处理时，我们可以使用流式AEAD原语**：

```java
AeadConfig.register();
KeysetHandle keysetHandle = KeysetHandle.generateNew(StreamingAeadKeyTemplates.AES128_CTR_HMAC_SHA256_4KB);
StreamingAead streamingAead = StreamingAeadFactory.getPrimitive(keysetHandle);

FileChannel cipherTextDestination = new FileOutputStream("cipherTextFile").getChannel();
WritableByteChannel encryptingChannel = streamingAead.newEncryptingChannel(cipherTextDestination, associatedData.getBytes());

ByteBuffer buffer = ByteBuffer.allocate(CHUNK_SIZE);
InputStream in = new FileInputStream("plainTextFile");

while (in.available() > 0) {
    in.read(buffer.array());
    encryptingChannel.write(buffer);
}

encryptingChannel.close();
in.close();
```

基本上，我们需要WriteableByteChannel来实现这一点。

因此，为了解密cipherTextFile，我们需要使用ReadableByteChannel：

```java
FileChannel cipherTextSource = new FileInputStream("cipherTextFile").getChannel();
ReadableByteChannel decryptingChannel = streamingAead.newDecryptingChannel(cipherTextSource, associatedData.getBytes());

OutputStream out = new FileOutputStream("plainTextFile");
int cnt = 1;
do {
    buffer.clear();
    cnt = decryptingChannel.read(buffer);
    out.write(buffer.array());
} while (cnt>0);

decryptingChannel.close();
out.close();
```

## 6. 混合加密

除了对称加密之外，Tink还实现了一些混合加密原语。

**通过混合加密我们可以获得对称密钥的效率和非对称密钥的便利性**。

简而言之，我们将使用对称密钥来加密明文，**并使用公钥仅加密对称密钥**。

请注意，它仅提供保密性，而不是发送者的身份真实性。

那么，让我们看看如何使用HybridEncrypt和HybridDecrypt：

```java
TinkConfig.register();

KeysetHandle privateKeysetHandle = KeysetHandle.generateNew(
  HybridKeyTemplates.ECIES_P256_HKDF_HMAC_SHA256_AES128_CTR_HMAC_SHA256);
KeysetHandle publicKeysetHandle = privateKeysetHandle.getPublicKeysetHandle();

String plaintext = "tuyucheng";
String contextInfo = "Tink";

HybridEncrypt hybridEncrypt = HybridEncryptFactory.getPrimitive(publicKeysetHandle);
HybridDecrypt hybridDecrypt = HybridDecryptFactory.getPrimitive(privateKeysetHandle);

byte[] ciphertext = hybridEncrypt.encrypt(plaintext.getBytes(), contextInfo.getBytes());
byte[] plaintextDecrypted = hybridDecrypt.decrypt(ciphertext, contextInfo.getBytes());
```

contextInfo是来自上下文的隐式公共数据，可以为空或为null，或者用作AEAD加密的“关联数据”输入或HKDF的“CtxInfo”输入。

密文允许检查contextInfo的完整性，但不能检查其保密性或真实性。

## 7. 消息认证码

**Tink还支持消息认证码或MAC**。

MAC是由几个字节组成的块，我们可以使用它来验证消息。

让我们看看如何创建MAC并验证其真实性：

```java
TinkConfig.register();

KeysetHandle keysetHandle = KeysetHandle.generateNew(MacKeyTemplates.HMAC_SHA256_128BITTAG);

String data = "tuyucheng";

Mac mac = MacFactory.getPrimitive(keysetHandle);

byte[] tag = mac.computeMac(data.getBytes());
mac.verifyMac(tag, data.getBytes());
```

如果数据不真实，方法verifyMac()将抛出GeneralSecurityException。

## 8. 数字签名

除了加密API，Tink还支持数字签名。

**为了实现数字签名，该库使用PublicKeySign原语对数据进行签名，并使用PublickeyVerify进行验证**：

```java
TinkConfig.register();

KeysetHandle privateKeysetHandle = KeysetHandle.generateNew(SignatureKeyTemplates.ECDSA_P256);
KeysetHandle publicKeysetHandle = privateKeysetHandle.getPublicKeysetHandle();

String data = "baeldung";

PublicKeySign signer = PublicKeySignFactory.getPrimitive(privateKeysetHandle);
PublicKeyVerify verifier = PublicKeyVerifyFactory.getPrimitive(publicKeysetHandle);

byte[] signature = signer.sign(data.getBytes()); 
verifier.verify(signature, data.getBytes());
```

与之前的加密方法类似，当签名无效时，我们会得到一个GeneralSecurityException。

## 9. 结论

在本文中，我们介绍了使用Java实现的Google Tink库。

我们已经了解了如何使用加密和解密数据以及如何保护其完整性和真实性。此外，我们还了解了如何使用数字签名API签署数据。