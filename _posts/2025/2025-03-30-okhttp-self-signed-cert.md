---
layout: post
title:  在OkHttp中信任自签名证书
category: libraries
copyright: libraries
excerpt: OkHttp
---

## 1. 概述

**在本文中，我们将了解如何初始化和配置OkHttpClient以信任自签名证书**。为此，我们将设置一个由自签名证书保护的最小HTTPS Spring Boot应用程序。

## 2. 基础知识

在深入研究实现此功能的代码之前，我们先来了解一下基本原理。**[SSL的本质](https://www.baeldung.com/java-ssl)是它在任意两方(通常是客户端和服务器)之间建立安全连接**，此外，它还有助于保护通过网络传输的数据的隐私性和完整性。

[JSSE API](https://docs.oracle.com/en/java/javase/11/security/java-secure-socket-extension-jsse-reference-guide.html)是Java SE的一个安全组件，为SSL/TLS协议提供了完整的API支持。

SSL证书(又称数字证书)在建立TLS握手、促进通信方之间的加密和信任方面发挥着至关重要的作用。自签名SSL证书不是由知名且受信任的证书颁发机构(CA)颁发的证书，开发人员可以轻松[生成和签名](https://www.baeldung.com/openssl-self-signed-cert)这些证书，以便为其软件启用HTTPS。

**由于自签名证书不可信，因此浏览器和标准HTTPS客户端(如[OkHttp](https://www.baeldung.com/guide-to-okhttp)和[Apache HTTP Client)](https://www.baeldung.com/httpclient-ssl)默认都不信任它们**。

最后，我们可以[使用Web浏览器或OpenSSL命令行实用程序方便地获取服务器证书](https://www.baeldung.com/linux/ssl-certificates)。

## 3. 设置测试环境

为了演示应用程序使用OkHttp接受和信任自签名证书，让我们快速配置并启动一个启用了HTTPS(由自签名证书保护)的简单Spring Boot应用程序。

默认配置将启动一个监听8443端口的Tomcat服务器，并公开一个可通过“https://localhost:8443/welcome”访问的安全REST API。

现在，让我们使用OkHttp客户端向该服务器发出HTTPS请求并使用“/welcome” API。

## 4. OkHttpClient和SSL

本节将初始化OkHttpClient并使用它来连接到我们刚刚设置的测试环境。此外，我们将检查路径中遇到的错误，并逐步实现使用OkHttp信任自签名证书的最终目标。

首先，让我们为OkHttpClient创建一个构建器：

```java
OkHttpClient.Builder builder = new OkHttpClient.Builder();
```

另外，让我们声明本教程中将使用的HTTPS URL：

```java
int SSL_APPLICATION_PORT = 8443;
String HTTPS_WELCOME_URL = "https://localhost:" + SSL_APPLICATION_PORT + "/welcome";
```

### 4.1 SSLHandshakeException

如果没有为SSL配置OkHttpClient，当我们尝试使用HTTPS URL时，就会出现安全异常：

```java
@Test(expected = SSLHandshakeException.class)
public void whenHTTPSSelfSignedCertGET_thenException() {
    builder.build()
            .newCall(new Request.Builder()
                    .url(HTTPS_WELCOME_URL).build())
            .execute();
}
```

堆栈跟踪如下：

```text
javax.net.ssl.SSLHandshakeException: PKIX path building failed: 
    sun.security.provider.certpath.SunCertPathBuilderException:
    unable to find valid certification path to requested target
    ...
```

上述错误恰恰意味着服务器使用了未经证书颁发机构(CA)签名的自签名证书。

因此，客户端无法验证直至根证书的[信任链](https://docs.oracle.com/cd/E19146-01/821-1828/ginal/index.html)，所以它抛出了[SSLHandshakeException](https://www.baeldung.com/java-ssl-handshake-failures)。

### 4.2 SSLPeerUnverifiedException

现在，让我们配置信任证书的OkHttpClient，无论其性质如何-CA签名或自签名。

首先，我们需要创建自己的[TrustManager](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/javax/net/ssl/X509TrustManager.html)，以使默认证书验证无效，并用我们的自定义实现覆盖这些验证：

```java
TrustManager TRUST_ALL_CERTS = new X509TrustManager() {
    @Override
    public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) {
    }

    @Override
    public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) {
    }

    @Override
    public java.security.cert.X509Certificate[] getAcceptedIssuers() {
        return new java.security.cert.X509Certificate[] {};
    }
};
```

接下来，我们将使用上面的TrustManager初始化[SSLContext](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/javax/net/ssl/SSLContext.html)，并设置OkHttpClient构建器的[SSLSocketFactory](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/javax/net/ssl/SSLSocketFactory.html)：

```java
SSLContext sslContext = SSLContext.getInstance("SSL");
sslContext.init(null, new TrustManager[] { TRUST_ALL_CERTS }, new java.security.SecureRandom());
builder.sslSocketFactory(sslContext.getSocketFactory(), (X509TrustManager) TRUST_ALL_CERTS);
```

再次，让我们运行测试。不难想象，即使经过上述调整，使用HTTPS URL仍会引发错误：

```java
@Test(expected = SSLPeerUnverifiedException.class)
public void givenTrustAllCerts_whenHTTPSSelfSignedCertGET_thenException() {
    // initializing the SSLContext and set the sslSocketFactory
    builder.build()
            .newCall(new Request.Builder()
                    .url(HTTPS_WELCOME_URL).build())
            .execute();
}
```

确切的错误是：

```text
javax.net.ssl.SSLPeerUnverifiedException: Hostname localhost not verified:
    certificate: sha256/bzdWeeiDwIVjErFX98l+ogWy9OFfBJsTRWZLB/bBxbw=
    DN: CN=localhost, OU=localhost, O=localhost, L=localhost, ST=localhost, C=IN
    subjectAltNames: []
```

这是由于一个众所周知的问题-[主机名验证失败](https://tersesystems.com/blog/2014/03/23/fixing-hostname-verification/)。**大多数HTTP库都会根据证书的SubjectAlternativeName的DNS名称字段执行主机名验证，而该字段在服务器的自签名证书中不可用**，如上图详细的堆栈跟踪所示。

### 4.3 覆盖HostnameVerifier

正确配置OkHttpClient的最后一步是禁用默认的HostnameVerifier，并将其替换为另一个绕过主机名验证的HostnameVerifier。

让我们添加最后一部分自定义：

```java
builder.hostnameVerifier(new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        return true;
    }
});
```

现在，让我们最后一次运行测试：

```java
@Test
public void givenTrustAllCertsSkipHostnameVerification_whenHTTPSSelfSignedCertGET_then200OK() {
    // initializing the SSLContext and set the sslSocketFactory
    // set the custom hostnameVerifier
    Response response = builder.build()
            .newCall(new Request.Builder()
                    .url(HTTPS_WELCOME_URL).build())
            .execute();
    assertEquals(200, response.code());
    assertNotNull(response.body());
    assertEquals("<h1>Welcome to Secured Site</h1>", response.body()
            .string());
}
```

**最后，OkHttpClient能够成功使用由自签名证书保护的HTTPS URL**。

## 5. 总结

在本教程中，我们学习了如何为OkHttpClient配置SSL，以便它能够信任自签名证书并使用任何HTTPS URL。

然而，需要考虑的一点是，尽管这种设计完全省略了证书验证和主机名验证，但客户端和服务器之间的所有通信仍然是加密的。双方之间的信任虽然丢失了，但SSL握手和加密并没有受到影响。