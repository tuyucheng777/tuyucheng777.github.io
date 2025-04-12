---
layout: post
title:  使用SSL的Apache HttpClient
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

本文将介绍如何**配置Apache HttpClient 4和5并使其支持“全部接受”SSL**，目标很简单：使用没有有效证书的HTTPS URL。

如果你想深入了解使用HttpClient可以做的其他有趣的事情-请直接转到[主要的HttpClient指南](https://www.baeldung.com/httpclient-guide)。

## 2. SSLPeerUnverifiedException 

如果不使用HttpClient配置SSL，则以下测试(使用HTTPS URL)将会失败：

```java
@Test
void whenHttpsUrlIsConsumed_thenException() {
    String urlOverHttps = "https://localhost:8082/httpclient-simple";
    HttpGet getMethod = new HttpGet(urlOverHttps);

    assertThrows(SSLPeerUnverifiedException.class, () -> {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        HttpResponse response = httpClient.execute(getMethod, new CustomHttpClientResponseHandler());
        assertThat(response.getCode(), equalTo(200));
    });
}
```

确切的错误是：

```text
javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
    at sun.security.ssl.SSLSessionImpl.getPeerCertificates(SSLSessionImpl.java:397)
    at org.apache.http.conn.ssl.AbstractVerifier.verify(AbstractVerifier.java:126)
    ...
```

只要无法为URL建立有效的信任链，就会发生[javax.net.ssl.SSLPeerUnverifiedException](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/javax/net/ssl/SSLPeerUnverifiedException.html)异常。

## 3. 配置SSL–全部接受(HttpClient 5)

现在让我们配置HTTP客户端来信任所有证书链，无论其有效性如何：

```java
@Test
void givenAcceptingAllCertificates_whenHttpsUrlIsConsumed_thenOk() throws GeneralSecurityException, IOException {
    final HttpGet getMethod = new HttpGet(HOST_WITH_SSL);

    final TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
    final SSLContext sslContext = SSLContexts.custom()
            .loadTrustMaterial(null, acceptingTrustStrategy)
            .build();
    final SSLConnectionSocketFactory sslsf =
            new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE);
    final Registry<ConnectionSocketFactory> socketFactoryRegistry =
            RegistryBuilder.<ConnectionSocketFactory> create()
                    .register("https", sslsf)
                    .register("http", new PlainConnectionSocketFactory())
                    .build();

    final BasicHttpClientConnectionManager connectionManager =
            new BasicHttpClientConnectionManager(socketFactoryRegistry);

    try( CloseableHttpClient httpClient = HttpClients.custom()
            .setConnectionManager(connectionManager)
            .build();

         CloseableHttpResponse response = (CloseableHttpResponse) httpClient
                 .execute(getMethod, new CustomHttpClientResponseHandler())) {

        final int statusCode = response.getCode();
        assertThat(statusCode, equalTo(HttpStatus.SC_OK));
    }
}
```

现在，新的TrustStrategy覆盖了标准证书验证过程(该过程应咨询已配置的信任管理器)——测试现已通过，**并且客户端能够使用HTTPS URL**。

## 4. 配置SSL–全部接受(HttpClient 4.5)

```java
@Test
public final void givenAcceptingAllCertificates_whenHttpsUrlIsConsumed_thenOk() throws GeneralSecurityException {
    TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
    SSLContext sslContext = SSLContexts.custom().loadTrustMaterial(null, acceptingTrustStrategy).build();
    SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext,
            NoopHostnameVerifier.INSTANCE);

    Registry<ConnectionSocketFactory> socketFactoryRegistry =
            RegistryBuilder.<ConnectionSocketFactory> create()
                    .register("https", sslsf)
                    .register("http", new PlainConnectionSocketFactory())
                    .build();

    BasicHttpClientConnectionManager connectionManager =
            new BasicHttpClientConnectionManager(socketFactoryRegistry);
    CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf)
            .setConnectionManager(connectionManager).build();

    HttpComponentsClientHttpRequestFactory requestFactory =
            new HttpComponentsClientHttpRequestFactory(httpClient);
    ResponseEntity<String> response = new RestTemplate(requestFactory)
            .exchange(urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode().value(), equalTo(200));
}
```

## 5. 使用SSL的Spring RestTemplate(HttpClient 5)

现在我们已经了解了如何配置具有SSL支持的原始HttpClient，让我们看一下更高级别的客户端-Spring RestTemplate。

如果没有配置SSL，以下测试将按预期失败：

```java
@Test
void whenHttpsUrlIsConsumed_thenException() {
    final String urlOverHttps = "https://localhost:8443/httpclient-simple/api/bars/1";

    assertThrows(ResourceAccessException.class, () -> {
        final ResponseEntity<String> response = new RestTemplate()
                .exchange(urlOverHttps, HttpMethod.GET, null, String.class);
        assertThat(response.getStatusCode().value(), equalTo(200));
    });
}
```

那么让我们配置SSL：

```java
@Test
void givenAcceptingAllCertificates_whenHttpsUrlIsConsumed_thenOk() throws GeneralSecurityException {
    final TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
    final SSLContext sslContext = SSLContexts.custom()
            .loadTrustMaterial(null, acceptingTrustStrategy)
            .build();
    final SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE);
    final Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory> create()
            .register("https", sslsf)
            .register("http", new PlainConnectionSocketFactory())
            .build();

    final BasicHttpClientConnectionManager connectionManager =
            new BasicHttpClientConnectionManager(socketFactoryRegistry);
    final CloseableHttpClient httpClient = HttpClients.custom()
            .setConnectionManager(connectionManager)
            .build();

    final HttpComponentsClientHttpRequestFactory requestFactory =
            new HttpComponentsClientHttpRequestFactory(httpClient);
    final ResponseEntity<String> response = new RestTemplate(requestFactory)
            .exchange(urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode()
            .value(), equalTo(200));
}
```

如你所见，**这与我们为原始HttpClient配置SSL的方式非常相似**-我们使用SSL支持配置请求工厂，然后通过这个预配置的工厂实例化模板。

## 6. 使用SSL的Spring RestTemplate(HttpClient 4.5)

```java
@Test
void givenAcceptingAllCertificates_whenUsingRestTemplate_thenCorrect() {
    final CloseableHttpClient httpClient = HttpClients.custom()
            .setSSLHostnameVerifier(new NoopHostnameVerifier())
            .build();
    final HttpComponentsClientHttpRequestFactory requestFactory
            = new HttpComponentsClientHttpRequestFactory();
    requestFactory.setHttpClient(httpClient);

    final ResponseEntity<String> response = new RestTemplate(requestFactory).exchange(urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode().value(), equalTo(200));
}
```

## 7. 总结

本教程讨论了如何为Apache HttpClient配置SSL，以便它能够使用任何HTTPS URL(无论证书是什么)；本教程还演示了Spring RestTemplate的相同配置。

然而，需要理解的一件重要事情是，这种策略完全忽略了证书检查-这使得它不安全，并且只能在有意义的地方使用。