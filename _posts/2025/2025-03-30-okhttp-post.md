---
layout: post
title:  使用OkHttp发送POST请求的快速指南
category: libraries
copyright: libraries
excerpt: OkHttp
---

## 1. 简介

我们在[OkHttp指南](https://www.baeldung.com/guide-to-okhttp)中介绍了[OkHttp](https://square.github.io/okhttp/)客户端的基础知识。

在这个简短的教程中，我们将特别介绍客户端3.x版本的不同类型的POST请求。

## 2. 基本POST

我们可以使用FormBody.Builder构建一个基本的RequestBody，通过POST请求发送两个参数-username和password：

```java
@Test
public void whenSendPostRequest_thenCorrect() throws IOException {
    RequestBody formBody = new FormBody.Builder()
            .add("username", "test")
            .add("password", "test")
            .build();

    Request request = new Request.Builder()
            .url(BASE_URL + "/users")
            .post(formBody)
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 3. 带授权的POST

如果我们想要验证请求，我们可以使用Credentials.basic构建器将凭据添加到标头。

在这个简单的例子中，我们还将发送一个字符串作为请求的主体：

```java
@Test
public void whenSendPostRequestWithAuthorization_thenCorrect() throws IOException {
    String postBody = "test post";
    
    Request request = new Request.Builder()
        .url(URL_SECURED_BY_BASIC_AUTHENTICATION)
        .addHeader("Authorization", Credentials.basic("username", "password"))
        .post(RequestBody.create(MediaType.parse("text/x-markdown), postBody))
        .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 4. 使用JSON进行POST

为了在请求主体中发送JSON，我们必须设置其媒体类型application/json，我们可以使用RequestBody.create构建器来实现这一点：

```java
@Test
public void whenPostJson_thenCorrect() throws IOException {
    String json = "{\"id\":1,\"name\":\"John\"}";

    RequestBody body = RequestBody.create(
            MediaType.parse("application/json"), json);

    Request request = new Request.Builder()
            .url(BASE_URL + "/users/detail")
            .post(body)
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 5. 多部分POST请求

我们将要看的最后一个例子是POST多部分请求，我们需要将RequestBody构建为MultipartBody来发布文件、用户名和密码：

```java
@Test
public void whenSendMultipartRequest_thenCorrect() throws IOException {
    RequestBody requestBody = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("username", "test")
            .addFormDataPart("password", "test")
            .addFormDataPart("file", "file.txt",
                    RequestBody.create(MediaType.parse("application/octet-stream"),
                            new File("src/test/resources/test.txt")))
            .build();

    Request request = new Request.Builder()
            .url(BASE_URL + "/users/multipart")
            .post(requestBody)
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 6. 使用非默认字符编码进行POST

OkHttp的默认字符编码是UTF-8：

```java
@Test
public void whenPostJsonWithoutCharset_thenCharsetIsUtf8() throws IOException {
    final String json = "{\"id\":1,\"name\":\"John\"}";

    final RequestBody body = RequestBody.create(MediaType.parse("application/json"), json);

    String charset = body.contentType().charset().displayName();

    assertThat(charset, equalTo("UTF-8"));
}
```

如果我们想使用不同的字符编码，我们可以将其作为MediaType.parse()的第二个参数传递：

```java
@Test
public void whenPostJsonWithUtf16Charset_thenCharsetIsUtf16() throws IOException {
    final String json = "{\"id\":1,\"name\":\"John\"}";

    final RequestBody body = RequestBody.create(MediaType.parse("application/json; charset=utf-16"), json);

    String charset = body.contentType().charset().displayName();

    assertThat(charset, equalTo("UTF-16"));
}
```

## 7. 总结

在这篇简短的文章中，我们看到了使用OkHttp客户端的几个POST请求的示例。