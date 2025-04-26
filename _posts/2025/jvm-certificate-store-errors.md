---
layout: post
title:  解决JVM中的证书存储错误
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本教程中，我们将了解发出SSL请求时可能遇到的常见问题。

## 2. 证书存储错误

每当Java应用程序与远程方建立SSL连接时，它都需要通过验证服务器[证书](https://www.baeldung.com/java-security-overview#public_key_infrastructure)来检查该服务器是否可信，**如果根证书未包含在证书存储文件中，则会引发安全异常**：

```text
Untrusted: Exception in thread "main" javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

我们需要记住，该文件的默认位置是$JAVA_HOME/lib/security/cacerts。

## 3. 自签名证书

在非生产环境中，**通常会发现由不受信任的颁发者签名的证书，这些证书称为自签名证书**。

我们可以在https://wrong.host.badssl.com/或https://self-signed.badssl.com/下找到一些不受信任证书的示例，在任何浏览器中打开这两个URL都会导致安全异常。我们可以检查一下，看看证书之间的差异。

打开https://self-signed.badssl.com/后，我们可以看到浏览器返回“Cert Authority Invalid”错误，因为该证书是由浏览器未知的颁发机构颁发的：

![](/assets/images/2025/javajvm/jvmcertificatestoreerrors01.png)

另一方面，打开https://wrong.host.badssl.com/会导致“Cert Common Name Invalid”错误，这种错误表明证书颁发给的主机名与提供的主机名不同：

![](/assets/images/2025/javajvm/jvmcertificatestoreerrors02.png)

**对于在JDK/JRE中运行的应用程序也会发生同样的情况，这些安全异常将阻止我们与那些不受信任的方建立SSL连接**。

## 4. 管理证书存储以信任我们的证书

幸运的是，**JDK/JRE提供了一个与证书存储交互的工具来管理其内容**，这个工具就是Keytool，可以在$JAVA_HOME/bin/keytool中找到。

重要提示：keytool需要密码才能与其交互，**默认密码是“changeit”**。

### 4.1 列出证书

要获取JVM证书存储中注册的所有证书的列表，我们需要发出以下命令：

```shell
keytool -list -keystore $JAVA_HOME/lib/security/cacerts
```

这将返回包含所有条目的列表，例如：

```shell
Your keystore contains 227 entries

Alias name: accvraiz1
Creation date: Apr 14, 2021
Entry type: trustedCertEntry

Owner: C=ES, O=ACCV, OU=PKIACCV, CN=ACCVRAIZ1
Issuer: C=ES, O=ACCV, OU=PKIACCV, CN=ACCVRAIZ1
....
```

### 4.2 添加证书

要手动将证书添加到该列表中，以便在我们发出SSL请求时对其进行验证，我们需要执行以下命令：

```shell
keytool -import -trustcacerts -file [certificate-file] -alias [alias] -keystore $JAVA_HOME/lib/security/cacerts
```

例如：

```shell
keytool -import -alias ss-badssl.com -keystore $JAVA_HOME/lib/security/cacerts -file ss-badssl.pem
```

### 4.3 自定义证书存储路径

如果以上方法均无效，则可能是我们的Java应用程序使用了其他证书存储，为了确保这一点，我们可以指定每次运行Java应用程序时要使用的证书存储：

```shell
java -Djavax.net.ssl.trustStore=CustomTrustStorePath ...
```

这样，我们就能确保它使用的是我们之前编辑过的证书存储，如果这没有帮助，我们还可以通过应用VM选项来调试SSL连接：

```shell
-Djavax.net.debug=all
```

## 5. 自动化脚本

总而言之，我们可以创建一个简单但方便的脚本来自动化整个过程：

```shell
#!/bin/sh
# cacerts.sh
/usr/bin/openssl s_client -showcerts -connect $1:443 </dev/null 2>/dev/null | /usr/bin/openssl x509 -outform PEM > /tmp/$1.pem
$JAVA_HOME/bin/keytool -import -trustcacerts -file /tmp/$1.pem -alias $1 -keystore $JAVA_HOME/lib/security/cacerts
rm /tmp/$1.pem
```

在脚本中，我们可以看到第一部分打开了与作为第一个参数传递的DNS的SSL连接，并请求其显示证书。之后，证书信息通过openssl进行传输，进行摘要处理并将其存储为PEM文件。

最后，我们将使用这个PEM文件来指示keytool将证书导入到cacerts文件中，并以DNS作为别名。

例如，我们可以尝试添加https://self-signed.badssl.com的证书：

```shell
cacerts.sh self-signed.badssl.com
```

运行之后，我们可以检查cacerts文件现在是否包含证书：

```shell
keytool -list -keystore $JAVA_HOME/lib/security/cacerts
```

最后，我们会看到新的证书：

```shell
#5: ObjectId: 2.5.29.32 Criticality=false
CertificatePolicies [
[CertificatePolicyId: [2.5.29.32.0]

Alias name: self-signed.badssl.com
Creation date: Oct 22, 2021
Entry type: trustedCertEntry

Owner: CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US
Issuer: CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US
Serial number: c9c0f0107cc53eb0
Valid from: Mon Oct 11 22:03:54 CEST 2021 until: Wed Oct 11 22:03:54 CEST 2023
Certificate fingerprints:
....
```

## 6. 手动添加证书

如果由于某种原因我们不想使用openssl，我们还可以使用浏览器提取证书并通过keytool添加它。

在基于Chromium的浏览器中，我们打开https://self-signed.badssl.com/之类的网站，并打开开发者工具(Windows和Linux系统按F12)，然后点击“Security”选项卡，最后点击“View Certificate”，证书信息将会显示出来：

![](/assets/images/2025/javajvm/jvmcertificatestoreerrors03.png)

让我们转到“Details”选项卡，点击“Export”按钮并保存，这就是我们的PEM文件：

![](/assets/images/2025/javajvm/jvmcertificatestoreerrors04.png)

最后，我们使用keytool导入它：

```shell
$JAVA_HOME/bin/keytool -import -trustcacerts -file CERTIFICATEFILE -alias ALIAS -keystore $JAVA_HOME/lib/security/cacerts
```

## 7. 总结

在本文中，我们了解了如何将自签名证书添加到我们的JDK/JRE证书存储区。现在，我们的Java应用程序在与包含这些证书的站点建立SSL连接时，都可以信任服务器端。