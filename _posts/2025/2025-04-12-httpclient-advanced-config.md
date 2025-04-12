---
layout: post
title:  高级Apache HttpClient配置
category: apache
copyright: apache
excerpt: Apache HttpClient 
---

## 1. 概述

在本文中，我们将研究[Apache HttpClient](https://hc.apache.org/httpcomponents-client-5.3.x/examples.html)库的高级用法。

我们将查看向HTTP请求添加自定义标头的示例，并了解如何配置客户端以通过代理服务器授权和发送请求。

我们将使用Wiremock来对HTTP服务器进行存根(stub)，如果你想了解更多关于Wiremock的信息，请查看[这篇文章](https://www.baeldung.com/introduction-to-wiremock)。

## 2. 带有自定义User-Agent标头的HTTP请求

假设我们要在HTTP GET请求中添加自定义User-Agent标头，User-Agent标头包含一个特征字符串，允许网络协议对等体识别应用程序类型、操作系统以及请求软件用户代理的软件供应商或软件版本。

在开始编写HTTP客户端之前，我们需要启动嵌入式Mock服务器：

```java
@Rule
public WireMockRule serviceMock = new WireMockRule(8089);
```

创建HttpGet实例时，我们可以简单地使用setHeader()方法将标头的名称及其值一起传递，该标头将被添加到HTTP请求中：

```java
String userAgent = "TuyuchengAgent/1.0"; 
HttpClient httpClient = HttpClients.createDefault();

HttpGet httpGet = new HttpGet("http://localhost:8089/detail");
httpGet.setHeader(HttpHeaders.USER_AGENT, userAgent);

HttpResponse response = httpClient.execute(httpGet);

assertEquals(response.getCode(), 200);
```

我们添加User-Agent标头并通过execute()方法发送该请求。

当发送GET请求到URL /detail时，如果标头User-Agent的值等于“TuyuchengAgent/1.0”，则serviceMock将返回200 HTTP响应码：

```java
serviceMock.stubFor(get(urlEqualTo("/detail"))
    .withHeader("User-Agent", equalTo(userAgent))
    .willReturn(aResponse().withStatus(200)));
```

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 3. 在POST请求主体中发送数据

通常，当我们执行HTTP POST方法时，我们希望传递一个实体作为请求主体，创建HttpPost对象实例时，我们可以使用setEntity()方法将主体添加到该请求中：

```java
String xmlBody = "<xml><id>1</id></xml>";
HttpClient httpClient = HttpClients.createDefault();
HttpPost httpPost = new HttpPost("http://localhost:8089/person");
httpPost.setHeader("Content-Type", "application/xml");

StringEntity xmlEntity = new StringEntity(xmlBody);
httpPost.setEntity(xmlEntity);

HttpResponse response = httpClient.execute(httpPost);

assertEquals(response.getCode(), 200);
```

我们创建一个StringEntity实例，其主体采用XML格式，务必将Content-Type标头设置为“application/xml”，以便将我们要发送的内容类型信息传递给服务器。当serviceMock收到带有XML主体的POST请求时，它会以状态码200 OK进行响应：

```java
serviceMock.stubFor(post(urlEqualTo("/person"))
    .withHeader("Content-Type", equalTo("application/xml"))
    .withRequestBody(equalTo(xmlBody))
    .willReturn(aResponse().withStatus(200)));
```

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 4. 通过代理服务器发送请求

通常，我们的Web服务可能位于执行一些附加逻辑、缓存静态资源等的代理服务器后面。当我们创建HTTP客户端并向实际服务发送请求时，我们不想在每个HTTP请求上都处理这个问题。

为了测试这种情况，我们需要启动另一个嵌入式Web服务器：

```java
@Rule
public WireMockRule proxyMock = new WireMockRule(8090);
```

使用两个嵌入式服务器，第一个实际服务位于8089端口上，而代理服务器正在监听8090端口。

我们配置HttpClient，通过创建一个以HttpHost实例代理作为参数的DefaultProxyRoutePlanner通过代理发送所有请求：

```java
HttpHost proxy = new HttpHost("localhost", 8090);
DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);
HttpClient httpclient = HttpClients.custom()
    .setRoutePlanner(routePlanner)
    .build();
```

我们的代理服务器将所有请求重定向到监听8090端口的实际服务，在测试的最后，我们验证请求是否通过代理发送到了实际服务：

```java
proxyMock.stubFor(get(urlMatching(".*"))
    .willReturn(aResponse().proxiedFrom("http://localhost:8089/")));

serviceMock.stubFor(get(urlEqualTo("/private"))
    .willReturn(aResponse().withStatus(200)));

assertEquals(response.getCode(), 200);
proxyMock.verify(getRequestedFor(urlEqualTo("/private")));
serviceMock.verify(getRequestedFor(urlEqualTo("/private")));
```

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 5. 配置HTTP客户端通过代理授权

扩展前面的示例，在某些情况下，代理服务器用于执行授权。在这样的配置中，代理可以授权所有请求，并将其传递给隐藏在代理后面的服务器。

我们可以配置HttpClient通过代理发送每个请求，以及用于执行授权过程的Authorization标头。

假设我们有一个代理服务器，只授权一个用户“username_admin”，密码为“secret_password”。

我们需要创建BasicCredentialsProvider实例，其中包含将通过代理授权的用户凭据，为了使HttpClient自动添加具有正确值的Authorization标头，我们需要创建一个包含凭据的HttpClientContext以及一个用于存储凭据的BasicAuthCache：

```java
HttpHost proxy = new HttpHost("localhost", 8090);
DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);
//Client credentials
CredentialsProvider credentialsProvider = CredentialsProviderBuilder.create()	
    .add(new AuthScope(proxy), "username_admin", "secret_password".toCharArray())	
    .build();

// Create AuthCache instance 
AuthCache authCache = new BasicAuthCache(); 
// Generate BASIC scheme object and add it to the local auth cache
BasicScheme basicAuth = new BasicScheme(); 
authCache.put(proxy, basicAuth); 
HttpClientContext context = HttpClientContext.create(); 
context.setCredentialsProvider(credentialsProvider); 
context.setAuthCache(authCache); 

HttpClient httpclient = HttpClients.custom() 
    .setRoutePlanner(routePlanner) 
    .setDefaultCredentialsProvider(credentialsProvider) 
    .build();
```

当我们设置好HttpClient后，向服务发出请求将会导致通过代理发送一个带有Authorization头的请求来执行授权过程，授权头会在每次请求中自动设置。

让我们对服务执行一个实际请求：

```java
HttpGet httpGet = new HttpGet("http://localhost:8089/private");
httpGet.setHeader("Authorization", StandardAuthScheme.BASIC);
HttpResponse response = httpclient.execute(httpGet, context);
```

使用我们的配置验证httpClient上的execute()方法确认请求通过带有Authorization标头的代理进行：

```java
proxyMock.stubFor(get(urlMatching("/private"))
    .willReturn(aResponse().proxiedFrom("http://localhost:8089/")));
serviceMock.stubFor(get(urlEqualTo("/private"))
    .willReturn(aResponse().withStatus(200)));

assertEquals(response.getCode(), 200);
proxyMock.verify(getRequestedFor(urlEqualTo("/private"))
  .withHeader("Authorization", containing("Basic")));
serviceMock.verify(getRequestedFor(urlEqualTo("/private")));
```

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 6. 总结

本文介绍如何配置Apache HttpClient来执行高级HTTP调用，我们了解了如何通过代理服务器发送请求以及如何通过代理进行授权。