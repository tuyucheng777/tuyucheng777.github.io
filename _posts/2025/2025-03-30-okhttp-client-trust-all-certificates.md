---
layout: post
title:  在OkHttp中信任所有证书
category: libraries
copyright: libraries
excerpt: OkHttp
---

## 1. 概述

在本教程中，我们将了解如何**创建和配置OkHttpClient来信任所有证书**。

请参阅我们[关于OkHttp](https://www.baeldung.com/tag/okhttp/)的文章，了解有关该库的更多详细信息。

## 2. Maven依赖

让我们首先将[OkHttp](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>5.0.0-alpha.12</version>
</dependency>
```

## 3. 使用普通的OkHttpClient

首先，我们获取一个标准的OkHttpClient对象并调用一个证书已过期的网页：

```java
OkHttpClient client = new OkHttpClient.Builder().build();
client.newCall(new Request.Builder().url("https://expired.badssl.com/").build()).execute();
```

堆栈跟踪输出将如下所示：

```text
sun.security.validator.ValidatorException: PKIX path validation failed: java.security.cert.CertPathValidatorException: validity check failed
```

现在，让我们看看当我们尝试使用自签名证书的另一个网站时收到的错误：

```text
sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

让我们尝试使用错误主机证书的网站：

```text
Hostname wrong.host.badssl.com not verified
```

如我们所见，默认情况下，**如果调用的站点证书不正确，OkHttpClient将抛出错误**。接下来，我们将了解如何创建和配置OkHttpClient以信任所有证书。

## 4. 设置OkHttpClient来信任所有证书

让我们创建包含单个X509TrustManager的TrustManager数组，通过覆盖其方法来禁用默认证书验证：

```java
TrustManager[] trustAllCerts = new TrustManager[]{
        new X509TrustManager() {
            @Override
            public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) {
            }

            @Override
            public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) {
            }

            @Override
            public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                return new java.security.cert.X509Certificate[]{};
            }
        }
};
```

我们将使用这个TrustManager数组来创建SSLContext：

```java
SSLContext sslContext = SSLContext.getInstance("SSL");
sslContext.init(null, trustAllCerts, new java.security.SecureRandom());
```

然后，我们将使用这个SSLContext来设置OkHttpClient构建器的SSLSocketFactory：

```java
OkHttpClient.Builder newBuilder = new OkHttpClient.Builder();
newBuilder.sslSocketFactory(sslContext.getSocketFactory(), (X509TrustManager) trustAllCerts[0]);
newBuilder.hostnameVerifier((hostname, session) -> true);
```

我们还将新Builder的HostnameVerifier设置为一个新的HostnameVerifier对象，该对象的验证方法始终返回true。

最后，我们可以得到一个新的OkHttpClient对象并再次调用证书有问题的站点，而不会出现任何错误：

```java
OkHttpClient newClient = newBuilder.build();
newClient.newCall(new Request.Builder().url("https://expired.badssl.com/").build()).execute();
```

## 5. 总结

在这篇简短的文章中，我们了解了如何创建和配置OkHttpClient来信任所有证书。当然，不建议信任所有证书。但是，在某些情况下我们可能需要它。