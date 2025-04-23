---
layout: post
title:  设置自定义Feign客户端超时
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Feign
---

## 1. 简介

[Spring Cloud Feign Client](https://www.baeldung.com/spring-cloud-openfeign)是一个方便的声明式REST客户端，我们使用它来实现微服务之间的通信。

在这个简短的教程中，我们将展示如何设置自定义Feign Client连接超时(全局和每个客户端)。

## 2. 默认设置

Feign Client的配置相当简单。

就超时而言，它允许我们配置读取超时和连接超时。连接超时是TCP握手所需的时间，而读取超时是从套接字读取数据所需的时间。

连接和读取超时默认分别为10秒和60秒。

## 3. 全局范围

我们可以通过application.yml文件中设置的feign.client.config.default属性来设置适用于应用程序中每个Feign客户端的连接和读取超时：

```yaml
feign:
    client:
        config:
            default:
                connectTimeout: 60000
                readTimeout: 10000
```

**这些值表示超时之前的毫秒数**。

## 4. 每个客户端

还可以**通过命名客户端来为每个特定客户端设置这些超时**：

```yaml
feign:
    client:
        config:
            FooClient:
                connectTimeout: 10000
                readTimeout: 20000
```

当然，我们可以毫无问题地列出全局设置和每个客户端覆盖。

## 5. 总结

在本教程中，我们解释了如何调整Feign Client的超时以及如何通过application.yml文件设置自定义值。