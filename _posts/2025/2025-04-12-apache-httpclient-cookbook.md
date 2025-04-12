---
layout: post
title:  Apache HttpClient手册
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

本指南通过各种示例和用例展示了如何使用Apache HttpClient。

我们将演示5.x和4.5版本的示例。

## 2. 食谱

### 2.1 版本5.x的示例

创建http客户端：

```java
CloseableHttpClient httpClient = HttpClientBuilder.create().build();
```

发送基本GET请求：

```java
httpClient.execute(new HttpGet("http://www.google.com"),
    response -> {
      //handle response
    }
);
```

获取HTTP响应的状态码：

```java
httpClient.execute(new HttpGet("http://www.google.com"), 
   response -> {
        assertThat(response.getCode()).isEqualTo(200);
        return response;
   }
);
```

获取响应的媒体类型：

```java
httpClient.execute(new HttpGet("http://www.google.com"),
    response -> {
        final String contentMimeType = ContentType.parse(response.getEntity().getContentType()).getMimeType();
        assertThat(contentMimeType).isEqualTo(ContentType.TEXT_HTML.getMimeType());
        return response;
    }
);
```

获取响应主体：

```java
httpClient.execute(new HttpGet("http://www.google.com"), 
    response -> {
        String bodyAsString = EntityUtils.toString(response.getEntity());
        assertThat(bodyAsString, notNullValue());
        return response;
    }
);
```

配置请求的超时时间：

```java
RequestConfig requestConfig = RequestConfig.custom() .setConnectionRequestTimeout(Timeout.ofMilliseconds(2000L)) .build();

request.setConfig(requestConfig);

httpClient.execute(request, response -> { //handle response }
```

在整个客户端上配置超时：

```java
ConnectionConfig connConfig = ConnectionConfig.custom()
    .setConnectTimeout(timeout, TimeUnit.MILLISECONDS)
    .setSocketTimeout(timeout, TimeUnit.MILLISECONDS)
    .build();

RequestConfig requestConfig = RequestConfig.custom()
    .setConnectionRequestTimeout(Timeout.ofMilliseconds(2000L))
    .build();

BasicHttpClientConnectionManager cm = new BasicHttpClientConnectionManager();
cm.setConnectionConfig(connConfig);

CloseableHttpClient httpClient = HttpClientBuilder.create()
    .setDefaultRequestConfig(requestConfig)
    .setConnectionManager(cm)
    .build();
```

发送POST请求：

```java
httpClient.execute(new HttpPost(SAMPLE_URL), 
    response -> {
        // handle response 
    }
);
```

向请求添加参数：

```java
HttpPost httpPost = new HttpPost(SAMPLE_POST_URL);

List<NameValuePair> params = new ArrayList<NameValuePair>();
params.add(new BasicNameValuePair("key1", "value1")); 
params.add(new BasicNameValuePair("key2", "value2")); 

httpPost.setEntity(new UrlEncodedFormEntity(params);
```

配置如何处理HTTP请求的重定向：

```java
CloseableHttpClient httpClient = HttpClientBuilder.create()
    .disableRedirectHandling()
    .build();
httpClient.execute(new HttpGet("http://t.co/I5YYd9tddw"), 
    response -> {
        assertThat(response.getCode(), equalTo(301));
        return response;
    }
);
```

配置请求的标头：

```java
HttpGet request = new HttpGet(SAMPLE_URL);
request.addHeader(HttpHeaders.ACCEPT, "application/xml");
httpClient.execute(request, 
    response -> {
        // handle response
    }
);
```

从响应中获取标头：

```java
httpClient.execute(new HttpGet(SAMPLE_URL), 
    response -> {
        Header[] headers = response.getHeaders(HttpHeaders.CONTENT_TYPE);
        assertThat(headers, not(emptyArray()));
    }
);
```

关闭/释放资源：

```java
HttpGet httpGet = new HttpGet(SAMPLE_GET_URL);
try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
    httpClient.execute(httpGet, resp -> {
            assertThat(resp.getCode()).isEqualTo(200);
            return resp;
    });
}
```

### 2.2 版本4.5的示例

创建HTTP客户端：

```java
CloseableHttpClient client = HttpClientBuilder.create().build();
```

发送基本GET请求：

```java
client.execute(new HttpGet("http://www.google.com"));
```

获取HTTP响应的状态码：

```java
CloseableHttpResponse response = client.execute(new HttpGet("http://www.google.com"));
assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
```

获取响应的媒体类型：

```java
CloseableHttpResponse response = client.execute(new HttpGet("http://www.google.com"));
String contentMimeType = ContentType.getOrDefault(response.getEntity()).getMimeType();
assertThat(contentMimeType, equalTo(ContentType.TEXT_HTML.getMimeType()));
```

获取响应主体：

```java
CloseableHttpResponse response = client.execute(new HttpGet("http://www.google.com"));
String bodyAsString = EntityUtils.toString(response.getEntity());
assertThat(bodyAsString, notNullValue());
```

配置请求的超时时间：

```java
RequestConfig requestConfig = RequestConfig.custom()
        .setConnectionRequestTimeout(1000)
        .setConnectTimeout(1000)
        .setSocketTimeout(1000)
        .build();
    HttpGet request = new HttpGet(SAMPLE_URL);
    request.setConfig(requestConfig);
    client.execute(request);
```

在整个客户端上配置超时：

```java
RequestConfig requestConfig = RequestConfig.custom()
    .setConnectionRequestTimeout(1000)
    .setConnectTimeout(1000)
    .setSocketTimeout(1000)
    .build();
HttpClientBuilder builder = HttpClientBuilder.create().setDefaultRequestConfig(requestConfig);

client = builder.build();
```

发送POST请求：

```java
client.execute(new HttpPost(SAMPLE_URL));
```

向请求添加参数：

```java
HttpPost httpPost = new HttpPost(SAMPLE_POST_URL); 

List<NameValuePair> params = new ArrayList<NameValuePair>();
params.add(new BasicNameValuePair("key1", "value1"));
params.add(new BasicNameValuePair("key2", "value2"));
 
httpPost.setEntity(new UrlEncodedFormEntity(params);
```

配置如何处理HTTP请求的重定向：

```java
CloseableHttpClient client = HttpClientBuilder.create()
    .disableRedirectHandling()
    .build();
CloseableHttpResponse response = client.execute(new HttpGet("http://t.co/I5YYd9tddw"));
assertThat(response.getStatusLine().getStatusCode(), equalTo(301));
```

配置请求的标头：

```java
HttpGet request = new HttpGet(SAMPLE_URL);
request.addHeader(HttpHeaders.ACCEPT, "application/xml");
response = client.execute(request);
```

从响应中获取标头：

```java
CloseableHttpResponse response = client.execute(new HttpGet(SAMPLE_URL));
Header[] headers = response.getHeaders(HttpHeaders.CONTENT_TYPE);
assertThat(headers, not(emptyArray()));
```

关闭/释放资源：

```java
try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
    HttpGet httpGet = new HttpGet(SAMPLE_URL);
    try (CloseableHttpResponse response = httpClient.execute(httpGet)) {
         // handle response;
         HttpEntity entity = response.getEntity();
         if (entity != null) {
              try (InputStream instream = entity.getContent()) {
                  // Process the input stream if needed
              }
         }
     }
}
```

## 3. 深入HttpClient

如果使用得当，HttpClient库是一个非常强大的工具，如果你想开始探索客户端可以做什么-请查看一些教程：

- [HttpClient–获取状态码](https://www.baeldung.com/httpclient-status-code)

- [HttpClient–设置自定义标头](https://www.baeldung.com/httpclient-custom-http-header)