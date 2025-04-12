---
layout: post
title:  如何在Apache HttpClient中设置TLS版本
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 简介

Apache HttpClient是一个低级、轻量级的客户端HTTP库，用于与HTTP服务器通信。在本教程中，我们将学习如何在使用[HttpClient](https://www.baeldung.com/httpclient-guide)时配置支持的传输层安全性(TLS)版本。我们将首先概述客户端和服务器之间TLS版本协商的工作原理；之后，我们将介绍在使用HttpClient时配置支持的TLS版本的三种不同方法；在第3部分中，我们将描述Apache HttpClient 4在TLS配置的静态设置方面有何不同。

## 2. Apache HttpClient 5

## 2.1 TLS版本协商

TLS是一种互联网协议，用于在双方之间提供安全可信的通信，它封装了HTTP等应用层协议。TLS协议自1999年首次发布以来，已经过多次修订。**因此，客户端和服务器在建立新连接时，首先就使用的TLS版本达成一致至关重要**。客户端和服务器在交换hello消息后，会达成TLS版本一致：

1. 客户端发送支持的TLS版本列表
2. 服务器选择一个并在响应中包含所选版本
3. 客户端和服务器使用所选版本继续建立连接

由于存在[降级攻击](https://en.wikipedia.org/wiki/Downgrade_attack)的风险，正确配置Web客户端支持的TLS版本至关重要。**请注意，为了使用最新版本的TLS(TLS 1.3)，我们必须使用Java 11或更高版本**。

## 2.2 静态设置TLS版本

### 2.2.1 HttpClientConnectionManager

首先，我们使用自定义的TLS配置创建一个连接管理器；然后，我们将此连接管理器设置为使用HttpClients.custom()创建的自定义ClosableHttpClient。

```java
final HttpClientConnectionManager cm = PoolingHttpClientConnectionManagerBuilder.create()
    .setDefaultTlsConfig(TlsConfig.custom()
        .setHandshakeTimeout(Timeout.ofSeconds(30))
        .setSupportedProtocols(TLS.V_1_2, TLS.V_1_3)
            .build())
    .build();

return HttpClients.custom().setConnectionManager(cm).build();
```

返回的Httpclient对象现在可以执行HTTP请求，通过显式设置支持的协议，客户端将仅支持通过TLS 1.2或TLS 1.3进行通信。 

### 2.2.2 Java运行时参数

或者，我们可以使用Java的https.protocols系统属性配置支持的TLS版本，这种方法避免了将值硬编码到应用程序代码中。相反，我们将配置HttpClient在建立连接时使用系统属性。HttpClient API提供了两种方法来实现这一点，第一种方法是通过HttpClients#createSystem：

```java
CloseableHttpClient httpClient = HttpClients.createSystem();
```

如果需要更多的客户端配置，我们可以使用构建器方法：

```java
CloseableHttpClient httpClient = HttpClients.custom().useSystemProperties().build();
```

这两个方法都告诉HttpClient在连接配置期间使用系统属性，这允许我们在应用程序运行时使用命令行参数设置所需的TLS版本，例如：

```shell
$ java -Dhttps.protocols=TLSv1.1,TLSv1.2,TLSv1.3 -jar webClient.jar
```

## 2.3 动态设置TLS版本

你还可以根据主机名和端口等连接详细信息设置TLS版本，我们将扩展SSLConnectionSocketFactory并重写prepareSocket方法，客户端在发起新连接之前会调用prepareSocket方法，**这将使我们能够根据每个连接决定使用哪种TLS协议**。你也可以启用对旧版TLS的支持，但前提是远程主机具有特定的子域：

```java
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(SSLContexts.createDefault()){

    @Override
    protected void prepareSocket(SSLSocket socket) {

        String hostname = socket.getInetAddress().getHostName();
        if (hostname.endsWith("internal.system.com")){
            socket.setEnabledProtocols(new String[] { "TLSv1", "TLSv1.1", "TLSv1.2", "TLSv1.3" });
        }
        else {
            socket.setEnabledProtocols(new String[] {"TLSv1.3"});
        }
    }
};
CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
```

在上面的示例中，prepareSocket方法首先获取SSLSocket将要连接的远程主机名。然后，**该主机名用于确定要启用的TLS协议**。现在，我们的HTTP客户端将对每个请求强制执行TLS 1.3，除非目标主机名的格式为.internal.example.com，通过在创建新的SSLSocket之前插入自定义逻辑，我们的应用程序现在可以自定义TLS通信细节。

## 3. Apache HttpClient 4

## 3.1 静态设置TLS版本

### 3.1.1 SSL连接套接字工厂

让我们使用HttpClients#custom构建器方法公开的HttpClientBuilder来自定义我们的HTTPClient配置，此构建器模式允许我们传入自己的SSLConnectionSocketFactory，它将使用所需的一组受支持的TLS版本进行实例化：

```java
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
    SSLContexts.createDefault(),
    new String[] { "TLSv1.2", "TLSv1.3" },
    null,
    SSLConnectionSocketFactory.getDefaultHostnameVerifier());

CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
```

返回的Httpclient对象现在可以执行HTTP请求了，通过在SSLConnectionSocketFactory构造函数中显式设置支持的协议，客户端将仅支持通过TLS 1.2或TLS 1.3进行通信。请注意，在Apache HttpClient 4.3之前的版本中，该类被称为SSLSocketFactory。

## 4. 总结

在本文中，我们研究了使用Apache HttpClient库时配置支持的TLS版本的三种不同方法，我们已经了解了如何为所有连接设置TLS版本，或者如何基于每个连接设置TLS版本。