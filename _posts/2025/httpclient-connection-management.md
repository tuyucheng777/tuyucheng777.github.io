---
layout: post
title:  Apache HttpClient连接管理
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

在本教程中，我们将介绍HttpClient 5中的连接管理基础知识。

我们将介绍如何使用BasicHttpClientConnectionManager和PoolingHttpClientConnectionManager来强制安全、符合协议且高效地使用HTTP连接。

## 2. BasicHttpClientConnectionManager用于低级、单线程连接

从HttpClient 4.3.3开始，BasicHttpClientConnectionManager是HTTP连接管理器的最简单实现。

我们使用它来创建和管理一次只有一个线程可以使用的单个连接。

### 2.1 获取低级连接的连接请求(HttpClientConnection)

```java
BasicHttpClientConnectionManager connMgr = new BasicHttpClientConnectionManager();	
HttpRoute route = new HttpRoute(new HttpHost("www.tuyucheng.com", 443));	
final LeaseRequest connRequest = connMgr.lease("some-id", route, null);
```

lease方法从管理器获取一个用于连接特定路由的连接池，route参数指定到目标主机的“代理跳数”路由，或者目标主机本身。

可以直接使用HttpClientConnection运行请求，但是，请记住，这种低级方法冗长且难以管理。低级连接可用于访问套接字和连接数据(例如超时和目标主机信息)，但对于标准执行，HttpClient是一个更易于使用的API。

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 3. 使用PoolingHttpClientConnectionManager获取和管理多线程连接池

ClientConnectionPoolManager维护一个ManagedHttpClientConnections池，能够处理来自多个执行线程的连接请求。该管理器可以打开的并发连接池默认大小为：每个路由或目标主机5个连接，总打开连接数25个。

首先，我们来看看如何在简单的HttpClient上设置这个连接管理器：

### 3.1 在HttpClient上设置PoolingHttpClientConnectionManager

```java
PoolingHttpClientConnectionManager poolingConnManager = new PoolingHttpClientConnectionManager();	
CloseableHttpClient client = HttpClients.custom()	
    .setConnectionManager(poolingConnManager)	
    .build();	
client.execute(new HttpGet("https://www.tuyucheng.com"));	
assertTrue(poolingConnManager.getTotalStats().getLeased() == 1);
```

接下来，我们看看运行在两个不同线程中的两个HttpClient如何使用同一个连接管理器：

### 3.2 使用两个HttpClient分别连接到一个目标主机

```java
HttpGet get1 = new HttpGet("https://www.tuyucheng.com");	
HttpGet get2 = new HttpGet("https://www.google.com");	
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();	
CloseableHttpClient client1 = HttpClients.custom()	
    .setConnectionManager(connManager)	
    .build();	
CloseableHttpClient client2 = HttpClients.custom()	
    .setConnectionManager(connManager)	
    .build();
MultiHttpClientConnThread thread1 = new MultiHttpClientConnThread(client1, get1);
MultiHttpClientConnThread thread2 = new MultiHttpClientConnThread(client2, get2); 
thread1.start(); 
thread2.start(); 
thread1.join(); 
thread2.join();
```

请注意，**我们使用了一个非常简单的自定义线程实现**。

### 3.3 自定义线程执行GET请求

```java
public class MultiHttpClientConnThread extends Thread {
    private CloseableHttpClient client;
    private HttpGet get;

    // standard constructors	
    public void run(){
        try {
            HttpEntity entity = client.execute(get).getEntity();
            EntityUtils.consume(entity);
        } catch (ClientProtocolException ex) {
        } catch (IOException ex) {
        }
    }
}
```

**注意EntityUtils.consume(entity)调用，这对于使用响应(实体)的全部内容是必要的，以便管理器可以将连接释放回池中**。

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 4. 配置连接管理器

池连接管理器的默认值选择得很好，但是，根据我们的用例，它们可能太小了。

那么，让我们看看如何配置：

- 连接总数
- 每个(任何)路由的最大连接数
- 每个特定路由的最大连接数

### 4.1 增加可打开和管理的连接数，使其超出默认限制

```java
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();
connManager.setMaxTotal(5);
connManager.setDefaultMaxPerRoute(4);
HttpHost host = new HttpHost("www.tuyucheng.com", 80);
connManager.setMaxPerRoute(new HttpRoute(host), 5);
```

让我们回顾一下API：

- setMaxTotal(int max)：设置最大打开连接数
- setDefaultMaxPerRoute(int max)：设置每个路由的最大并发连接数，默认为2
- setMaxPerRoute(int max)：设置特定路由的并发连接总数，默认为2

因此，如果不改变默认值，**我们很容易就会达到连接管理器的极限**。

让我们看看它是什么样子的。

### 4.2 使用线程执行连接

```java
HttpGet get = new HttpGet("http://www.tuyucheng.com");	
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();	
CloseableHttpClient client = HttpClients.custom()	
    .setConnectionManager(connManager)	
    .build();	
MultiHttpClientConnThread thread1 = new MultiHttpClientConnThread(client, get, connManager);	
MultiHttpClientConnThread thread2 = new MultiHttpClientConnThread(client, get, connManager);	
MultiHttpClientConnThread thread3 = new MultiHttpClientConnThread(client, get, connManager);	
MultiHttpClientConnThread thread4 = new MultiHttpClientConnThread(client, get, connManager);	
MultiHttpClientConnThread thread5 = new MultiHttpClientConnThread(client, get, connManager);	
MultiHttpClientConnThread thread6 = new MultiHttpClientConnThread(client, get, connManager);	
thread1.start();	
thread2.start();	
thread3.start();	
thread4.start();	
thread5.start();	
thread6.start();	
thread1.join();	
thread2.join();	
thread3.join();	
thread4.join();	
thread5.join();	
thread6.join();
```

请记住，每个主机的连接数默认限制为5个。因此，在本例中，我们希望6个线程向同一主机发出6个请求，但并行只会分配3个连接。

让我们看一下日志。

我们有6个线程正在运行，但只有5个租用的连接：

```text
15:37:02.631 [Thread-0] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Leased Connections = 0	
15:37:02.631 [Thread-5] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Leased Connections = 0	
15:37:02.631 [Thread-1] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Leased Connections = 0	
15:37:02.631 [Thread-3] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Leased Connections = 0	
15:37:02.633 [Thread-5] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Available Connections = 0	
15:37:02.633 [Thread-1] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Available Connections = 0	
15:37:02.631 [Thread-4] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Leased Connections = 0	
15:37:02.631 [Thread-2] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Leased Connections = 0	
15:37:02.633 [Thread-4] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Available Connections = 0	
15:37:02.633 [Thread-0] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Available Connections = 0	
15:37:02.633 [Thread-3] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Available Connections = 0	
15:37:02.633 [Thread-2] INFO  c.t.t.h.conn.MultiHttpClientConnThread - Before - Available Connections = 0	
15:37:02.949 [Thread-1] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Leased Connections = 5	
15:37:02.949 [Thread-2] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Leased Connections = 5	
15:37:02.949 [Thread-5] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Leased Connections = 5	
15:37:02.949 [Thread-2] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Available Connections = 5	
15:37:02.949 [Thread-3] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Leased Connections = 5	
15:37:02.949 [Thread-1] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Available Connections = 5	
15:37:02.949 [Thread-5] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Available Connections = 5	
15:37:02.949 [Thread-3] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Available Connections = 5	
15:37:02.953 [Thread-0] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Leased Connections = 5	
15:37:02.953 [Thread-0] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Available Connections = 5	
15:37:03.004 [Thread-4] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Leased Connections = 1	
15:37:03.004 [Thread-4] INFO  c.t.t.h.conn.MultiHttpClientConnThread - After - Available Connections = 5
```

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 5. 连接保持策略

根据HttpClient 5.2：“如果响应中不存在Keep-Alive标头，则HttpClient假定连接可以保持3分钟。”

为了解决这个问题并能够管理死连接，我们需要一个定制的策略实现并将其构建到HttpClient中。

### 5.1 自定义Keep-Alive策略

```java
final ConnectionKeepAliveStrategy myStrategy = new ConnectionKeepAliveStrategy() {
    @Override
    public TimeValue getKeepAliveDuration(HttpResponse response, HttpContext context) {
        Args.notNull(response, "HTTP response");
        final Iterator<HeaderElement> it = MessageSupport.iterate(response, HeaderElements.KEEP_ALIVE);
        final HeaderElement he = it.next();
        final String param = he.getName();
        final String value = he.getValue();
        if (value != null && param.equalsIgnoreCase("timeout")) {
            try {
                return TimeValue.ofSeconds(Long.parseLong(value));
            } catch (final NumberFormatException ignore) {
            }
        }
        return TimeValue.ofSeconds(5);
    }
};
```

此策略将首先尝试应用标头中声明的主机Keep-Alive策略，如果响应标头中不存在该信息，则它将保持连接5秒钟。

现在让我们用这个自定义策略创建一个客户端：

```java
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom()
    .setKeepAliveStrategy(myStrategy)
    .setConnectionManager(connManager)
    .build();
```

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 6. 连接持久性/重用

HTTP/1.1规范规定，如果连接尚未关闭，我们可以重用它们，这称为连接持久性。

一旦管理器释放连接，它将保持开放状态以供重复使用。

当使用只能管理单个连接的BasicHttpClientConnectionManager时，必须先释放该连接，然后才能再次租回：

### 6.1 BasicHttpClientConnectionManager连接重用

```java
BasicHttpClientConnectionManager connMgr = new BasicHttpClientConnectionManager();	
HttpRoute route = new HttpRoute(new HttpHost("www.tuyucheng.com", 443));	
final HttpContext context = new BasicHttpContext();	
final LeaseRequest connRequest = connMgr.lease("some-id", route, null);	
final ConnectionEndpoint endpoint = connRequest.get(Timeout.ZERO_MILLISECONDS);	
connMgr.connect(endpoint, Timeout.ZERO_MILLISECONDS, context);	
connMgr.release(endpoint, null, TimeValue.ZERO_MILLISECONDS);	
CloseableHttpClient client = HttpClients.custom()	
    .setConnectionManager(connMgr)	
    .build();	
HttpGet httpGet = new HttpGet("https://www.example.com");	
client.execute(httpGet, context, response -> response);
```

让我们看看到底发生了什么。

请注意，我们首先使用低级连接，这样我们就可以完全控制何时释放连接，然后使用HttpClient进行正常的高级连接。

复杂的底层逻辑在这里不太重要，我们唯一关心的是releaseConnection调用，它会释放唯一可用的连接并允许其被重用。

然后客户端再次运行GET请求并成功。

如果我们跳过释放连接，我们将从HttpClient获得IllegalStateException：

```text
java.lang.IllegalStateException: Connection is still allocated
  at o.a.h.u.Asserts.check(Asserts.java:34)
  at o.a.h.i.c.BasicHttpClientConnectionManager.getConnection
    (BasicHttpClientConnectionManager.java:248)
```

请注意，现有连接并未关闭，只是被释放，然后由第二个请求重新使用。

与上面的例子相反，PoolingHttpClientConnectionManager允许透明地重用连接，而无需隐式释放连接：

### 6.2 PoolingHttpClientConnectionManager：使用线程重用连接

```java
HttpGet get = new HttpGet("http://www.tuyucheng.com");	
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();	
connManager.setDefaultMaxPerRoute(6);	
connManager.setMaxTotal(6);	

CloseableHttpClient client = HttpClients.custom()	
    .setConnectionManager(connManager)	
    .build();	

MultiHttpClientConnThread[] threads = new MultiHttpClientConnThread[10];	
for (int i = 0; i < threads.length; i++) {	
    threads[i] = new MultiHttpClientConnThread(client, get, connManager);	
}	
for (MultiHttpClientConnThread thread : threads) {	
    thread.start();	
}	
for (MultiHttpClientConnThread thread : threads) {	
    thread.join(1000);	
}
```

上面的例子有10个线程运行10个请求，但只共享6个连接。

当然，此示例依赖于服务器的Keep-Alive超时。为了确保连接在重用之前不会断开，我们应该为客户端配置Keep-Alive策略(参见示例5.1)。

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 7. 配置超时-使用连接管理器设置套接字超时

配置连接管理器时我们可以设置的唯一超时是套接字超时：

### 7.1 将套接字超时设置为5秒

```java
final HttpRoute route = new HttpRoute(new HttpHost("www.tuyucheng.com", 80));	
final HttpContext context = new BasicHttpContext();	
final PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();	
final ConnectionConfig connConfig = ConnectionConfig.custom()	
    .setSocketTimeout(5, TimeUnit.SECONDS)	
    .build();	
connManager.setDefaultConnectionConfig(connConfig);	
final LeaseRequest leaseRequest = connManager.lease("id1", route, null);	
final ConnectionEndpoint endpoint = leaseRequest.get(Timeout.ZERO_MILLISECONDS);	
connManager.connect(endpoint, null, context);	
connManager.close();
```

有关HttpClient中超时的更深入讨论，请参见[此处](https://www.baeldung.com/httpclient-timeout)。

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 8. 连接驱逐

我们使用连接驱逐功能来检测空闲和过期的连接并关闭它们。我们有两种方法可以做到这一点：

1. HttpClient提供evictExpiredConnections()，它使用后台线程主动从连接池中驱逐过期的连接。
2. HttpClient提供evictIdleConnections(final TimeValue maxIdleTime)，它使用后台线程主动从连接池中驱逐理想连接。

### 8.1 设置HttpClient检查并清除过期连接

```java
final PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();	
connManager.setMaxTotal(100);	
try (final CloseableHttpClient httpclient = HttpClients.custom()	
    .setConnectionManager(connManager)	
    .evictExpiredConnections()	
    .evictIdleConnections(TimeValue.ofSeconds(2))	
    .build()) {	
    // create an array of URIs to perform GETs on	
    final String[] urisToGet = { "http://hc.apache.org/", "http://hc.apache.org/httpcomponents-core-ga/"};	
    for (final String requestURI : urisToGet) {	
        final HttpGet request = new HttpGet(requestURI);	
        System.out.println("Executing request " + request.getMethod() + " " + request.getRequestUri());	
        httpclient.execute(request, response -> {	
            System.out.println("----------------------------------------");	
            System.out.println(request + "->" + new StatusLine(response));	
            EntityUtils.consume(response.getEntity());	
            return null;	
        });	
    }	
    final PoolStats stats1 = connManager.getTotalStats();	
    System.out.println("Connections kept alive: " + stats1.getAvailable());	
    // Sleep 4 sec and let the connection evict or do its job	
    Thread.sleep(4000);	
    final PoolStats stats2 = connManager.getTotalStats();	
    System.out.println("Connections kept alive: " + stats2.getAvailable());
}
```

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 9. 连接关闭

我们可以正常关闭连接(我们尝试在关闭之前刷新输出缓冲区)，或者我们可以通过调用关闭方法强制执行此操作(不刷新输出缓冲区)。

为了正确关闭连接，我们需要执行以下所有操作：

- 使用并关闭响应(如果可关闭)

- 关闭客户端

- 关闭连接管理器

### 9.1 关闭连接并释放资源

```java
PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();	
CloseableHttpClient client = HttpClients.custom()	
    .setConnectionManager(connManager)	
    .build();	
final HttpGet get = new HttpGet("http://google.com");	
CloseableHttpResponse response = client.execute(get);	
EntityUtils.consume(response.getEntity());	
response.close();	
client.close();	
connManager.close();
```

如果我们在关闭管理器之前没有关闭连接，则所有连接都将被关闭，所有资源将被释放。

重要的是要记住，这不会刷新现有连接中可能正在进行的任何数据。

有关4.5版本的相关Javadoc，请检查此[链接](https://javadoc.io/doc/org.apache.httpcomponents/httpclient/latest/index.html)和总结部分的Github链接。

## 10. 总结

在本文中，我们讨论了如何使用HttpClient的HTTP连接管理API来处理管理连接的整个过程，这包括打开和分配连接、管理多个代理的并发使用以及最终关闭连接。

我们了解了BasicHttpClientConnectionManager如何成为处理单个连接的简单解决方案以及它如何管理低级连接。

我们还了解了PoolingHttpClientConnectionManager如何与HttpClient API结合以实现高效且符合协议的HTTP连接使用。