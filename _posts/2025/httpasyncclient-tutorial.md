---
layout: post
title:  Apache HttpAsyncClient教程
category: apache
copyright: apache
excerpt: Apache HttpAsyncClient
---

## 1. 概述

在本教程中，我们将说明Apache HttpAsyncClient的最常见用例-从基本用法到如何设置代理、如何使用SSL证书以及最后如何使用异步客户端进行身份验证。

## 2. 简单示例

首先，让我们看看如何在一个简单的示例中使用HttpAsyncClient发送一个GET请求：

```java
@Test
void whenUseHttpAsyncClient_thenCorrect() throws InterruptedException, ExecutionException, IOException {
    final SimpleHttpRequest request = SimpleRequestBuilder.get(HOST_WITH_COOKIE)
            .build();
    final CloseableHttpAsyncClient client = HttpAsyncClients.custom()
            .build();
    client.start();


    final Future<SimpleHttpResponse> future = client.execute(request, null);
    final HttpResponse response = future.get();

    assertThat(response.getCode(), equalTo(200));
    client.close();
}
```

请注意，我们需要在使用异步客户端之前启动它；否则，我们将得到以下异常：

```text
java.lang.IllegalStateException: Request cannot be executed; I/O reactor status: INACTIVE
    at o.a.h.u.Asserts.check(Asserts.java:46)
    at o.a.h.i.n.c.CloseableHttpAsyncClientBase.
      ensureRunning(CloseableHttpAsyncClientBase.java:90)
```

## 3. 使用HttpAsyncClient实现多线程

现在让我们看看如何使用HttpAsyncClient同时执行多个请求。

在以下示例中，我们使用HttpAsyncClient向3个不同的主机发送3个GET请求。

## 3.1 对于HttpAsyncClient 5.x

```java
@Test
void whenUseMultipleHttpAsyncClient_thenCorrect() throws Exception {
    final IOReactorConfig ioReactorConfig = IOReactorConfig
            .custom()
            .build();

    final CloseableHttpAsyncClient client = HttpAsyncClients.custom()
            .setIOReactorConfig(ioReactorConfig)
            .build();

    client.start();
    final String[] toGet = { "http://www.google.com/", "http://www.apache.org/", "http://www.bing.com/" };

    final GetThread[] threads = new GetThread[toGet.length];
    for (int i = 0; i < threads.length; i++) {
        final HttpGet request = new HttpGet(toGet[i]);
        threads[i] = new GetThread(client, request);
    }

    for (final GetThread thread : threads) {
        thread.start();
    }

    for (final GetThread thread : threads) {
        thread.join();
    }
}
```

下面是我们处理响应的GetThread实现：

```java
static class GetThread extends Thread {

    private final CloseableHttpAsyncClient client;
    private final HttpContext context;
    private final HttpGet request;

    GetThread(final CloseableHttpAsyncClient client, final HttpGet request) {
        this.client = client;
        context = HttpClientContext.create();
        this.request = request;
    }

    @Override
    public void run() {
        try {
            final Future<HttpResponse> future = client.execute(request, context, null);
            final HttpResponse response = future.get();
            assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
        } catch (final Exception ex) {
            System.out.println(ex.getLocalizedMessage());
        }
    }
}
```

## 3.2 对于HttpAsyncClient4.5

```java
@Test
void whenUseMultipleHttpAsyncClient_thenCorrect() throws Exception {
    final ConnectingIOReactor ioReactor = new DefaultConnectingIOReactor();
    final PoolingNHttpClientConnectionManager cm = new PoolingNHttpClientConnectionManager(ioReactor);
    final CloseableHttpAsyncClient client = HttpAsyncClients.custom().setConnectionManager(cm).build();
    client.start();
    final String[] toGet = { "http://www.google.com/", "http://www.apache.org/", "http://www.bing.com/" };

    final GetThread[] threads = new GetThread[toGet.length];
    for (int i = 0; i < threads.length; i++) {
        final HttpGet request = new HttpGet(toGet[i]);
        threads[i] = new GetThread(client, request);
    }

    for (final GetThread thread : threads) {
        thread.start();
    }

    for (final GetThread thread : threads) {
        thread.join();
    }
}
```

## 4. 使用HttpAsyncClient代理

让我们看看如何使用HttpAsyncClient设置和使用代理。

在下面的示例中，我们通过代理发送HTTP GET请求。

## 4.1 对于HttpAsyncClient 5.x

```java
@Test
void whenUseProxyWithHttpClient_thenCorrect() throws Exception {
    final HttpHost proxy = new HttpHost("127.0.0.1", GetRequestMockServer.serverPort);
    DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);
    final CloseableHttpAsyncClient client = HttpAsyncClients.custom()
            .setRoutePlanner(routePlanner)
            .build();
    client.start();

    final SimpleHttpRequest request = new SimpleHttpRequest("GET" ,HOST_WITH_PROXY);
    final Future<SimpleHttpResponse> future = client.execute(request, null);
    final HttpResponse  response = future.get();
    assertThat(response.getCode(), equalTo(200));
    client.close();
}
```

## 4.2 对于HttpAsyncClient 4.5

```java
@Test
void whenUseProxyWithHttpClient_thenCorrect() throws Exception {
    final CloseableHttpAsyncClient client = HttpAsyncClients.createDefault();
    client.start();
    final HttpHost proxy = new HttpHost("127.0.0.1", GetRequestMockServer.serverPort);
    final RequestConfig config = RequestConfig.custom().setProxy(proxy).build();
    final HttpGet request = new HttpGet(HOST_WITH_PROXY);
    request.setConfig(config);
    final Future<HttpResponse> future = client.execute(request, null);
    final HttpResponse response = future.get();
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 5. 使用HttpAsyncClient的SSL证书

现在，让我们看看如何使用SSL证书和HttpAsyncClient。

在以下示例中，我们配置HttpAsyncClient以接收所有证书：

## 5.1 对于HttpAsyncClient 5.x

```java
@Test
void whenUseSSLWithHttpAsyncClient_thenCorrect() throws Exception {
    final TrustStrategy acceptingTrustStrategy = (certificate, authType) -> true;

    final SSLContext sslContext = SSLContexts.custom()
            .loadTrustMaterial(null, acceptingTrustStrategy)
            .build();

    final TlsStrategy tlsStrategy = ClientTlsStrategyBuilder.create()
            .setHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .setSslContext(sslContext)
            .build();

    final PoolingAsyncClientConnectionManager cm = PoolingAsyncClientConnectionManagerBuilder.create()
            .setTlsStrategy(tlsStrategy)
            .build();

    final CloseableHttpAsyncClient client = HttpAsyncClients.custom()
            .setConnectionManager(cm)
            .build();

    client.start();

    final SimpleHttpRequest request = new SimpleHttpRequest("GET",HOST_WITH_SSL);
    final Future<SimpleHttpResponse> future = client.execute(request, null);

    final HttpResponse response = future.get();
    assertThat(response.getCode(), equalTo(200));
    client.close();
}
```

## 5.2 对于HttpAsyncClient 4.5

```java
@Test
void whenUseSSLWithHttpAsyncClient_thenCorrect() throws Exception {
    final TrustStrategy acceptingTrustStrategy = (certificate, authType) -> true;
    final SSLContext sslContext = SSLContexts.custom()
            .loadTrustMaterial(null, acceptingTrustStrategy)
            .build();

    final CloseableHttpAsyncClient client = HttpAsyncClients.custom()
            .setSSLHostnameVerifier(NoopHostnameVerifier.INSTANCE)
            .setSSLContext(sslContext).build();

    client.start();
    final HttpGet request = new HttpGet(HOST_WITH_SSL);
    final Future<HttpResponse> future = client.execute(request, null);
    final HttpResponse response = future.get();
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 6. 在HttpAsyncClient中使用Cookie

接下来，让我们看看如何在HttpAsyncClient中使用Cookie。

在下面的示例中，我们在发送请求之前设置了一个Cookie值。

## 6.1 对于HttpAsyncClient 5.x

```java
@Test
void whenUseCookiesWithHttpAsyncClient_thenCorrect() throws Exception {
    final BasicCookieStore cookieStore = new BasicCookieStore();
    final BasicClientCookie cookie = new BasicClientCookie(COOKIE_NAME, "1234");
    cookie.setDomain(COOKIE_DOMAIN);
    cookie.setPath("/");
    cookieStore.addCookie(cookie);
    final CloseableHttpAsyncClient client = HttpAsyncClients.custom().build();
    client.start();
    final SimpleHttpRequest request = new SimpleHttpRequest("GET" ,HOST_WITH_COOKIE);

    final HttpContext localContext = new BasicHttpContext();
    localContext.setAttribute(HttpClientContext.COOKIE_STORE, cookieStore);

    final Future<SimpleHttpResponse> future = client.execute(request, localContext, null);

    final HttpResponse response = future.get();
    assertThat(response.getCode(), equalTo(200));
    client.close();
}
```

## 6.2 对于HttpAsyncClient 4.5

```java
@Test
void whenUseCookiesWithHttpAsyncClient_thenCorrect() throws Exception {
    final BasicCookieStore cookieStore = new BasicCookieStore();
    final BasicClientCookie cookie = new BasicClientCookie(COOKIE_NAME, "1234");
    cookie.setDomain(COOKIE_DOMAIN);
    cookie.setPath("/");
    cookieStore.addCookie(cookie);
    final CloseableHttpAsyncClient client = HttpAsyncClients.custom().build();
    client.start();
    final HttpGet request = new HttpGet(HOST_WITH_COOKIE);

    final HttpContext localContext = new BasicHttpContext();
    localContext.setAttribute(HttpClientContext.COOKIE_STORE, cookieStore);

    final Future<HttpResponse> future = client.execute(request, localContext, null);
    final HttpResponse response = future.get();
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 7. 使用HttpAsyncClient进行身份验证

接下来，让我们看看如何使用HttpAsyncClient进行身份验证。

在下面的示例中，我们使用CredentialsProvider通过基本身份验证访问主机。

## 7.1 对于HttpAsyncClient 5.x

```java
@Test
void whenUseAuthenticationWithHttpAsyncClient_thenCorrect() throws Exception {
    final BasicCredentialsProvider credsProvider = new BasicCredentialsProvider();
    final UsernamePasswordCredentials credentials =
        new UsernamePasswordCredentials(DEFAULT_USER, DEFAULT_PASS.toCharArray());
    credsProvider.setCredentials(new AuthScope(URL_SECURED_BY_BASIC_AUTHENTICATION, 80) ,credentials);
    final CloseableHttpAsyncClient client = HttpAsyncClients
        .custom()
        .setDefaultCredentialsProvider(credsProvider).build();

    final SimpleHttpRequest request = new SimpleHttpRequest("GET" ,URL_SECURED_BY_BASIC_AUTHENTICATION);

    client.start();

    final Future<SimpleHttpResponse> future = client.execute(request, null);

    final HttpResponse response = future.get();
    assertThat(response.getCode(), equalTo(200));
    client.close();
}
```

## 7.2 对于HttpAsyncClient 4.5

```java
@Test
public void whenUseAuthenticationWithHttpAsyncClient_thenCorrect() throws Exception {
    CredentialsProvider provider = new BasicCredentialsProvider();
    UsernamePasswordCredentials creds = new UsernamePasswordCredentials("user", "pass");
    provider.setCredentials(AuthScope.ANY, creds);
    
    CloseableHttpAsyncClient client = 
      HttpAsyncClients.custom().setDefaultCredentialsProvider(provider).build();
    client.start();
    
    HttpGet request = new HttpGet("http://localhost:8080");
    Future<HttpResponse> future = client.execute(request, null);
    HttpResponse response = future.get();
    
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 8. 总结

在本文中，我们阐述了异步Apache Http客户端的各种用例。