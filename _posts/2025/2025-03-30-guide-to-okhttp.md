---
layout: post
title:  OkHttp指南
category: libraries
copyright: libraries
excerpt: OkHttp
---

## 1. 简介

在本教程中，我们将探索发送不同类型的HTTP请求以及接收和解释HTTP响应的基础知识。然后我们将学习如何使用[OkHttp](http://square.github.io/okhttp/)配置客户端。

最后，我们将讨论使用自定义标头、超时、响应缓存等配置客户端的更高级用例。

## 2. OkHttp概述

OkHttp是适用于Android和Java应用程序的高效HTTP和HTTP/2客户端。

它具有一些高级功能，例如连接池(如果HTTP/2不可用)、透明GZIP压缩和响应缓存，以完全避免网络重复请求。

它还能够从常见的连接问题中恢复；当连接失败时，如果服务有多个IP地址，它可以向备用地址重试请求。

从高层次上看，客户端既设计用于阻塞同步调用，也设计用于非阻塞异步调用。

OkHttp支持Android 2.3及以上版本。对于Javalin，最低要求为1.7。

现在我们已经给出了简要的概述，让我们看一些使用示例。

## 3. Maven依赖

首先，我们将该库作为依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>5.0.0-alpha.12</version>
</dependency>
```

要查看此库的最新依赖，请查看[Maven Central](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)上的页面。

## 4. 使用OkHttp进行同步GET

要发送同步GET请求，我们需要根据URL构建一个Request对象并发送Call。执行后，我们将得到一个Response实例：

```java
@Test
public void whenGetRequest_thenCorrect() throws IOException {
    Request request = new Request.Builder()
            .url(BASE_URL + "/date")
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 5. 使用OkHttp进行异步GET

要进行异步GET，我们需要将Call放入队列。Callback允许我们在响应可读时读取响应，这发生在响应标头准备就绪之后。

读取响应主体可能仍会被阻塞，OkHttp目前不提供任何异步API来分部分接收响应主体：

```java
@Test
public void whenAsynchronousGetRequest_thenCorrect() {
    Request request = new Request.Builder()
            .url(BASE_URL + "/date")
            .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        public void onResponse(Call call, Response response) throws IOException {
            // ...
        }

        public void onFailure(Call call, IOException e) {
            fail();
        }
    });
}
```

## 6. 带查询参数的GET

最后，为了向我们的GET请求添加查询参数，我们可以利用HttpUrl.Builder。

构建URL后，我们可以将其传递给我们的Request对象：

```java
@Test
public void whenGetRequestWithQueryParameter_thenCorrect() throws IOException {
    HttpUrl.Builder urlBuilder
            = HttpUrl.parse(BASE_URL + "/ex/bars").newBuilder();
    urlBuilder.addQueryParameter("id", "1");

    String url = urlBuilder.build().toString();

    Request request = new Request.Builder()
            .url(url)
            .build();
    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

## 7. POST请求 

现在让我们看一个简单的POST请求，其中我们构建一个RequestBody来发送参数“username”和“password”：

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

我们的文章[使用OkHttp进行POST请求的快速指南](https://www.baeldung.com/okhttp-post)有更多使用OkHttp进行POST请求的示例。

## 8. 文件上传

### 8.1 上传文件

在此示例中，我们将演示如何上传文件，我们将使用MultipartBody.Builder上传“test.ext”文件：

```java
@Test
public void whenUploadFile_thenCorrect() throws IOException {
    RequestBody requestBody = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("file", "file.txt",
                    RequestBody.create(MediaType.parse("application/octet-stream"),
                            new File("src/test/resources/test.txt")))
            .build();

    Request request = new Request.Builder()
            .url(BASE_URL + "/users/upload")
            .post(requestBody)
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

### 8.2 获取文件上传进度

然后我们将学习如何获取文件上传的进度，我们将扩展RequestBody以了解上传过程。

上传方法如下：

```java
@Test
public void whenGetUploadFileProgress_thenCorrect() throws IOException {
    RequestBody requestBody = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("file", "file.txt",
                    RequestBody.create(MediaType.parse("application/octet-stream"),
                            new File("src/test/resources/test.txt")))
            .build();

    ProgressRequestWrapper.ProgressListener listener
            = (bytesWritten, contentLength) -> {
        float percentage = 100f * bytesWritten / contentLength;
        assertFalse(Float.compare(percentage, 100) > 0);
    };

    ProgressRequestWrapper countingBody
            = new ProgressRequestWrapper(requestBody, listener);

    Request request = new Request.Builder()
            .url(BASE_URL + "/users/upload")
            .post(countingBody)
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

现在这里是ProgressListener接口，它使我们能够观察上传进度：

```java
public interface ProgressListener {
    void onRequestProgress(long bytesWritten, long contentLength);
}
```

接下来是ProgressRequestWrapper，它是RequestBody的扩展版本：

```java
public class ProgressRequestWrapper extends RequestBody {

    @Override
    public void writeTo(BufferedSink sink) throws IOException {
        BufferedSink bufferedSink;

        countingSink = new CountingSink(sink);
        bufferedSink = Okio.buffer(countingSink);

        delegate.writeTo(bufferedSink);

        bufferedSink.flush();
    }
}
```

最后，这是CountingSink，它是ForwardingSink的扩展版本：

```java
protected class CountingSink extends ForwardingSink {

    private long bytesWritten = 0;

    public CountingSink(Sink delegate) {
        super(delegate);
    }

    @Override
    public void write(Buffer source, long byteCount)
            throws IOException {
        super.write(source, byteCount);

        bytesWritten += byteCount;
        listener.onRequestProgress(bytesWritten, contentLength());
    }
}
```

注意：

- 当将ForwardingSink扩展为“CountingSink”时，我们重写了write()方法来计算写入(传输)的字节数
- 在将RequestBody扩展到“ProgressRequestWrapper”时，我们重写了writeTo()方法以使用我们的“ForwardingSink”

## 9. 设置自定义标头

### 9.1 设置请求的标头

要在请求上设置任何自定义标头，我们可以使用简单的addHeader调用：

```java
@Test
public void whenSetHeader_thenCorrect() throws IOException {
    Request request = new Request.Builder()
            .url(SAMPLE_URL)
            .addHeader("Content-Type", "application/json")
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();
    response.close();
}
```

### 9.2 设置默认标头

在这个例子中，我们将看到如何在客户端本身上配置默认标头，而不是在每个请求上都设置它。

例如，如果我们想为每个请求设置一个内容类型“application/json”，我们需要为我们的客户端设置一个拦截器：

```java
@Test
public void whenSetDefaultHeader_thenCorrect() throws IOException {
    OkHttpClient client = new OkHttpClient.Builder()
            .addInterceptor(
                    new DefaultContentTypeInterceptor("application/json"))
            .build();

    Request request = new Request.Builder()
            .url(SAMPLE_URL)
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();
    response.close();
}
```

这是DefaultContentTypeInterceptor，它是Interceptor的扩展版本：

```java
public class DefaultContentTypeInterceptor implements Interceptor {

    public Response intercept(Interceptor.Chain chain) throws IOException {
        Request originalRequest = chain.request();
        Request requestWithUserAgent = originalRequest
                .newBuilder()
                .header("Content-Type", contentType)
                .build();

        return chain.proceed(requestWithUserAgent);
    }
}
```

请注意，拦截器将标头添加到原始请求中。

## 10. 不遵循重定向

在这个例子中，我们将看到如何配置OkHttpClient以停止遵循重定向。

默认情况下，如果GET请求以HTTP 301 Moved Permanently响应，则会自动进行重定向。在某些用例中，这完全没问题，但在其他用例中则不需要这样做。

为了实现这种行为，当我们构建客户端时，我们需要将followRedirects设置为false。

请注意，响应将返回HTTP 301状态码：

```java
@Test
public void whenSetFollowRedirects_thenNotRedirected() throws IOException {
    OkHttpClient client = new OkHttpClient().newBuilder()
            .followRedirects(false)
            .build();

    Request request = new Request.Builder()
            .url("http://t.co/I5YYd9tddw")
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(301));
}
```

如果我们使用true参数打开重定向(或完全删除它)，客户端将遵循重定向，并且测试将失败，因为返回代码将是HTTP 200。

## 11. 超时

当对端无法访问时，我们可以使用超时来使调用失败。网络故障可能是由于客户端连接问题、服务器可用性问题或其他任何原因造成的。OkHttp支持连接、读取和写入超时。

在这个例子中，我们构建了客户端，其readTimeout为1秒，而URL的提供有2秒的延迟：

```java
@Test
public void whenSetRequestTimeout_thenFail() throws IOException {
    OkHttpClient client = new OkHttpClient.Builder()
            .readTimeout(1, TimeUnit.SECONDS)
            .build();

    Request request = new Request.Builder()
            .url(BASE_URL + "/delay/2")
            .build();

    Call call = client.newCall(request);
    Response response = call.execute();

    assertThat(response.code(), equalTo(200));
}
```

请注意，由于客户端超时低于资源响应时间，测试将失败。

## 12. 取消调用

我们可以使用Call.cancel()立即停止正在进行的调用，如果线程当前正在写入请求或读取响应，则会抛出IOException。

当不再需要调用时，我们使用此方法来节省网络，例如当用户离开应用程序时：

```java
@Test(expected = IOException.class)
public void whenCancelRequest_thenCorrect() throws IOException {
    ScheduledExecutorService executor
            = Executors.newScheduledThreadPool(1);

    Request request = new Request.Builder()
            .url(BASE_URL + "/delay/2")
            .build();

    int seconds = 1;
    long startNanos = System.nanoTime();

    Call call = client.newCall(request);

    executor.schedule(() -> {
        logger.debug("Canceling call: "
                + (System.nanoTime() - startNanos) / 1e9f);

        call.cancel();

        logger.debug("Canceled call: "
                + (System.nanoTime() - startNanos) / 1e9f);

    }, seconds, TimeUnit.SECONDS);

    logger.debug("Executing call: "
            + (System.nanoTime() - startNanos) / 1e9f);

    Response response = call.execute();

    logger.debug("Call was expected to fail, but completed: "
            + (System.nanoTime() - startNanos) / 1e9f, response);
}
```

## 13. 响应缓存

要创建缓存，我们需要一个可以读写的缓存目录，以及缓存大小的限制。

客户端将使用它来缓存响应：

```java
@Test
public void  whenSetResponseCache_thenCorrect() throws IOException {
    int cacheSize = 10 * 1024 * 1024;

    File cacheDirectory = new File("src/test/resources/cache");
    Cache cache = new Cache(cacheDirectory, cacheSize);

    OkHttpClient client = new OkHttpClient.Builder()
            .cache(cache)
            .build();

    Request request = new Request.Builder()
            .url("http://publicobject.com/helloworld.txt")
            .build();

    Response response1 = client.newCall(request).execute();
    logResponse(response1);

    Response response2 = client.newCall(request).execute();
    logResponse(response2);
}
```

启动测试后，第一次调用的响应不会被缓存。对方法cacheResponse的调用将返回null，而对方法networkResponse的调用将返回来自网络的响应。

缓存文件夹也将被缓存文件填满。

第二次调用执行将产生相反的效果，因为响应已被缓存。这意味着对networkResponse的调用将返回null，而对cacheResponse的调用将返回缓存中的响应。

为了防止响应使用缓存，我们可以使用CacheControl.FORCE_NETWORK。为了防止它使用网络，我们可以使用CacheControl.FORCE_CACHE。

值得注意的是，如果我们使用FORCE_CACHE，并且响应需要网络，OkHttp将返回504 Unsatisfiable Request响应。

## 14. 总结

在本文中，我们探讨了如何使用OkHttp作为HTTP和HTTP/2客户端的几个示例。