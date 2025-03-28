---
layout: post
title:  Java版BouncyCastle简介
category: libraries
copyright: libraries
excerpt: BouncyCastle
---

## 1. 概述

[BouncyCastle](https://www.bouncycastle.org/)是一个Java库，是对默认Java加密扩展(JCE)的补充。

在这篇介绍性文章中，我们将展示如何使用BouncyCastle执行加密操作，例如加密和签名。

## 2. Maven配置

在开始使用该库之前，我们需要将所需的依赖项添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcpkix-jdk18on</artifactId>
    <version>1.76</version>
</dependency>
```

请注意，我们始终可以在[Maven Central](https://mvnrepository.com/artifact/org.bouncycastle)中查找最新的依赖版本。

## 3. 设置无限强度管辖权策略文件

标准Java安装在加密功能强度方面受到限制，这是因为策略禁止使用超过某些值(例如AES的128)的密钥。

为了克服这个限制，**我们需要配置无限强度的管辖策略文件**。

为此，我们首先需要通过以下[链接](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)下载软件包。之后，我们需要将压缩文件解压到我们选择的目录中-其中包含两个jar文件：

- local_policy.jar
- US_export_policy.jar

最后，我们需要查找{JAVA_HOME}/lib/security文件夹并用我们在此处提取的策略文件替换现有的策略文件。

请注意，**在Java 9中，我们不再需要下载策略软件包**，将crypto.policy属性设置为unlimited就足够了：

```java
Security.setProperty("crypto.policy", "unlimited");
```

完成后，我们需要检查配置是否正常工作：

```java
int maxKeySize = javax.crypto.Cipher.getMaxAllowedKeyLength("AES");
System.out.println("Max Key Size for AES : " + maxKeySize);
```

因此：

```text
Max Key Size for AES : 2147483647
```

根据getMaxAllowedKeyLength()方法返回的最大密钥大小，我们可以有把握地说，无限强度策略文件已正确安装。

如果返回值等于128，我们需要确保已将文件安装到运行代码的JVM中。

## 4. 加密操作

### 4.1 准备证书和私钥

在我们开始实现加密函数之前，我们首先需要创建一个证书和一个私钥。

为了测试目的，我们可以使用这些资源：

- [Tuyucheng.cer](https://github.com/eugenp/tutorials/tree/master/libraries/src/main/resources)
- [Tuyucheng.p12](https://github.com/eugenp/tutorials/tree/master/libraries/src/main/resources)

Tuyucheng.cer是使用国际X.509公钥基础设施标准的数字证书，而Tuyucheng.p12是包含私钥的受密码保护的[PKCS12密钥库](https://tools.ietf.org/html/rfc7292)。

让我们看看如何在Java中加载它们：

```java
Security.addProvider(new BouncyCastleProvider());
CertificateFactory certFactory= CertificateFactory
    .getInstance("X.509", "BC");
 
X509Certificate certificate = (X509Certificate) certFactory
    .generateCertificate(new FileInputStream("Tuyucheng.cer"));
 
char[] keystorePassword = "password".toCharArray();
char[] keyPassword = "password".toCharArray();
 
KeyStore keystore = KeyStore.getInstance("PKCS12");
keystore.load(new FileInputStream("Tuyucheng.p12"), keystorePassword);
PrivateKey key = (PrivateKey) keystore.getKey("Tuyucheng", keyPassword);
```

首先，我们使用addProvider()方法动态添加BouncyCastleProvider作为安全提供程序。

也可以通过编辑{JAVA_HOME}/jre/lib/security/java.security文件并添加以下行来静态完成此操作：

```text
security.provider.N = org.bouncycastle.jce.provider.BouncyCastleProvider
```

一旦正确安装了提供程序，我们就使用getInstance()方法创建一个CertificationFactory对象。

getInstance()方法接收两个参数；证书类型“X.509”和安全提供程序“BC”。

随后，通过generateCertificate()方法使用certFactory实例生成X509Certificate对象。

以同样的方式，我们创建了一个PKCS12 Keystore对象，并调用了该对象的load()方法。

getKey()方法返回与给定别名关联的私钥。

请注意，PKCS12 Keystore包含一组私钥，每个私钥可以有一个特定的密码，这就是为什么我们需要一个全局密码来打开Keystore，以及一个特定密码来检索私钥。

证书和私钥对主要用于非对称加密操作：

- 加密
- 解密
- 签名
- 确认

### 4.2 生成证书和密钥库

如果我们想要生成另一个证书，我们将必须生成另一个与该证书链接的KeyStore。

要生成证书，我们需要运行以下命令：

1. 生成长度为2048的私钥：
   `openssl genrsa -out private-key.pem 2048`
2. 使用该私钥生成证书签名请求：
   `openssl req -new -sha256 -key private-key.pem -out certificate-signed-request.csr`
3. 生成适用于Web服务器的自签名证书：
   `openssl req -x509 -sha256 -days 365 -key private-key.pem -in certificate-signed-request.csr -out Tuyucheng.cer`
4. 生成PKCS12格式的Keystore：
   `openssl pkcs12 -export -name tuyucheng -out Tuyucheng.p12 -inkey private-key.pem -in Tuyucheng.cer`

生成证书成功后，将证书添加到资源文件夹，请确保代码中引用了正确的证书和KeyStore名称。

### 4.3 CMS/PKCS7加密和解密

**在非对称加密密码学中，每次通信都需要一个公共证书和一个私钥**。

收件人绑定到所有发件人之间公开共享的证书。

简而言之，发送者需要收件人的证书来加密消息，而收件人需要相关的私钥才能解密。

让我们看看如何使用加密证书实现encryptData()函数：

```java
public static byte[] encryptData(byte[] data, X509Certificate encryptionCertificate)
        throws CertificateEncodingException, CMSException, IOException {

   byte[] encryptedData = null;
   if (null != data && null != encryptionCertificate) {
      CMSEnvelopedDataGenerator cmsEnvelopedDataGenerator
              = new CMSEnvelopedDataGenerator();

      JceKeyTransRecipientInfoGenerator jceKey
              = new JceKeyTransRecipientInfoGenerator(encryptionCertificate);
      cmsEnvelopedDataGenerator.addRecipientInfoGenerator(transKeyGen);
      CMSTypedData msg = new CMSProcessableByteArray(data);
      OutputEncryptor encryptor
              = new JceCMSContentEncryptorBuilder(CMSAlgorithm.AES128_CBC)
              .setProvider("BC").build();
      CMSEnvelopedData cmsEnvelopedData = cmsEnvelopedDataGenerator
              .generate(msg,encryptor);
      encryptedData = cmsEnvelopedData.getEncoded();
   }
   return encryptedData;
}
```

我们使用收件人的证书创建了一个JceKeyTransRecipientInfoGenerator对象。

然后，我们创建了一个新的CMSEnvelopedDataGenerator对象并将收件人信息生成器添加到其中。

之后，我们使用JceCMSContentEncryptorBuilder类创建一个OutputEncryptor对象，使用AES CBC算法。

加密器稍后用于生成封装加密消息的CMSEnvelopedData对象。

最后，信封的编码表示形式以字节数组的形式返回。

现在，让我们看看decryptData()方法的实现是什么样的：

```java
public static byte[] decryptData(
        byte[] encryptedData,
        PrivateKey decryptionKey)
        throws CMSException {

   byte[] decryptedData = null;
   if (null != encryptedData && null != decryptionKey) {
      CMSEnvelopedData envelopedData = new CMSEnvelopedData(encryptedData);

      Collection<RecipientInformation> recipients
              = envelopedData.getRecipientInfos().getRecipients();
      KeyTransRecipientInformation recipientInfo
              = (KeyTransRecipientInformation) recipients.iterator().next();
      JceKeyTransRecipient recipient
              = new JceKeyTransEnvelopedRecipient(decryptionKey);

      return recipientInfo.getContent(recipient);
   }
   return decryptedData;
}
```

首先，我们使用加密数据字节数组初始化CMSEnvelopedData对象，然后使用getRecipients()方法检索消息的所有预期收件人。

完成后，我们创建了一个与收件人的私钥关联的新JceKeyTransRecipient对象。

RecipientInfo实例包含解密/封装的消息，但是除非我们拥有相应收件人的密钥，否则我们无法检索它。

**最后，给定收件人密钥作为参数，getContent()方法返回从与该收件人关联的EnvelopedData中提取的原始字节数组**。

让我们编写一个简单的测试来确保一切都按预期运行：

```java
String secretMessage = "My password is 123456Seven";
System.out.println("Original Message : " + secretMessage);
byte[] stringToEncrypt = secretMessage.getBytes();
byte[] encryptedData = encryptData(stringToEncrypt, certificate);
System.out.println("Encrypted Message : " + new String(encryptedData));
byte[] rawData = decryptData(encryptedData, privateKey);
String decryptedMessage = new String(rawData);
System.out.println("Decrypted Message : " + decryptedMessage);
```

因此：

```text
Original Message : My password is 123456Seven
Encrypted Message : 0�*�H��...
Decrypted Message : My password is 123456Seven
```

### 4.4 CMS/PKCS7签名与验证

签名和验证是验证数据真实性的加密操作。

让我们看看如何使用数字证书签署秘密消息：

```java
public static byte[] signData(
        byte[] data,
        X509Certificate signingCertificate,
        PrivateKey signingKey) throws Exception {

   byte[] signedMessage = null;
   List<X509Certificate> certList = new ArrayList<X509Certificate>();
   CMSTypedData cmsData= new CMSProcessableByteArray(data);
   certList.add(signingCertificate);
   Store certs = new JcaCertStore(certList);

   CMSSignedDataGenerator cmsGenerator = new CMSSignedDataGenerator();
   ContentSigner contentSigner
           = new JcaContentSignerBuilder("SHA256withRSA").build(signingKey);
   cmsGenerator.addSignerInfoGenerator(new JcaSignerInfoGeneratorBuilder(
           new JcaDigestCalculatorProviderBuilder().setProvider("BC")
                   .build()).build(contentSigner, signingCertificate));
   cmsGenerator.addCertificates(certs);

   CMSSignedData cms = cmsGenerator.generate(cmsData, true);
   signedMessage = cms.getEncoded();
   return signedMessage;
}
```

首先，我们将输入嵌入到CMSTypedData中，然后创建一个新的CMSSignedDataGenerator对象。

我们使用SHA256withRSA作为签名算法，并使用我们的签名密钥创建了一个新的ContentSigner对象。

随后使用contentSigner实例与签名证书一起创建SigningInfoGenerator对象。

将SignerInfoGenerator和签名证书添加到CMSSignedDataGenerator实例后，我们最终使用generate()方法创建一个CMS签名数据对象，该对象也带有CMS签名。

现在我们已经了解了如何签名数据，让我们看看如何验证签名的数据：

```java
public static boolean verifySignedData(byte[] signedData) throws Exception {
   X509Certificate signCert = null;
   ByteArrayInputStream inputStream
           = new ByteArrayInputStream(signedData);
   ASN1InputStream asnInputStream = new ASN1InputStream(inputStream);
   CMSSignedData cmsSignedData = new CMSSignedData(
           ContentInfo.getInstance(asnInputStream.readObject()));

   SignerInformationStore signers
           = cmsSignedData.getCertificates().getSignerInfos();
   SignerInformation signer = signers.getSigners().iterator().next();
   Collection<X509CertificateHolder> certCollection
           = certs.getMatches(signer.getSID());
   X509CertificateHolder certHolder = certCollection.iterator().next();

   return signer
           .verify(new JcaSimpleSignerInfoVerifierBuilder()
                   .build(certHolder));
}
```

再次，我们根据签名的数据字节数组创建了一个CMSSignedData对象，然后，我们使用getSignerInfos()方法检索与签名相关的所有签名者。

在此示例中，我们仅验证了一个签名者，但对于通用用途，必须遍历getSigners()方法返回的签名者集合并分别检查每个签名者。

最后，我们使用build()方法创建了一个SignerInformationVerifier对象并将其传递给verify()方法。

如果给定的对象可以成功验证签名者对象上的签名，则verify()方法返回true。

这是一个简单的例子：

```java
byte[] signedData = signData(rawData, certificate, privateKey);
Boolean check = verifSignData(signedData);
System.out.println(check);
```

因此：

```text
true
```

## 5. 总结

在本文中，我们探讨了如何使用BouncyCastle库执行基本的加密操作，例如加密和签名。

在现实世界中，我们经常希望对数据进行签名然后加密，这样，只有接收者才能使用私钥对其进行解密，并根据数字签名检查其真实性。