---
layout: post
title:  Ratpack与Hystrix
category: webmodules
copyright: webmodules
excerpt: Ratpack
---

## 1. 简介

之前，我们已经[展示](https://www.baeldung.com/ratpack)了如何使用Ratpack构建高性能、响应灵敏的应用程序。

在本文中，我们将研究如何将Netflix Hystrix与Ratpack应用程序集成。

Netflix Hystrix通过隔离访问点来阻止级联故障并提供容错回退选项，从而帮助控制分布式服务之间的交互，它可以帮助我们构建更具弹性的应用程序。请参阅我们对[Hystrix的介绍](https://www.baeldung.com/introduction-to-hystrix)以进行快速回顾。

这就是我们的使用方法-利用Hystrix提供的这些有用功能来增强我们的Ratpack应用程序。

**注意：ratpack-hystrix模块已被删除：https://github.com/ratpack/ratpack/blob/master/release-notes.md?plain=1，因此本文中的代码不再维护**。

## 2. Maven依赖

要将Hystrix与Ratpack一起使用，我们需要项目pom.xml中的ratpack-hystrix依赖：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-hystrix</artifactId>
    <version>1.4.6</version>
</dependency>
```

ratpack-hystrix的最新版本可以在[这里](https://mvnrepository.com/artifact/io.ratpack/ratpack-hystrix)找到，ratpack-hystrix包括[ratpack-core](https://mvnrepository.com/artifact/io.ratpack/ratpack-core)和[hystrix-core](https://mvnrepository.com/artifact/com.netflix.hystrix/hystrix-core)。

为了利用Ratpack的响应式功能，我们还需要ratpack-rx：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-rx</artifactId>
    <version>1.4.6</version>
</dependency>
```

你可以在[此处](https://mvnrepository.com/artifact/io.ratpack/ratpack-rx)找到ratpack-rx的最新版本。

## 3. 使用Hystrix命令提供服务

使用Hystrix时，底层服务一般被包装在[HystrixCommand](https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html)或者[HystrixObservableCommand](https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixObservableCommand.html)中，Hystrix支持以同步、异步、响应式的方式执行这些命令。其中，只有响应式是非阻塞的，也是官方推荐的。

**在以下示例中，我们将构建一些从[Github REST API](https://docs.github.com/en/rest)获取Profile的端点**。

### 3.1 响应式命令执行

首先，让我们使用Hystrix构建一个响应式后端服务：

```java
public class HystrixReactiveHttpCommand extends HystrixObservableCommand<String> {

    //...

    @Override
    protected Observable<String> construct() {
        return RxRatpack.observe(httpClient
                .get(uri, r -> r.headers(h -> h.add("User-Agent", "Tuyucheng HttpClient")))
                .map(res -> res.getBody().getText()));
    }

    @Override
    protected Observable<String> resumeWithFallback() {
        return Observable.just("tuyucheng's reactive fallback profile");
    }
}
```

这里使用Ratpack响应式[HttpClient](https://hc.apache.org/httpcomponents-client-5.3.x/index.html)发出GET请求，HystrixReactiveHttpCommand可以充当响应式处理程序：

```java
chain.get("rx", ctx -> 
    new HystrixReactiveHttpCommand(
        ctx.get(HttpClient.class), eugenGithubProfileUri, timeout)
        .toObservable()
        .subscribe(ctx::render));
```

可以使用以下测试来验证端点：

```java
@Test
public void whenFetchReactive_thenGotEugenProfile() {
    assertThat(appUnderTest.getHttpClient().getText("rx"), containsString("www.tuyucheng.com"));
}
```

### 3.2 异步命令执行

HystrixCommand的异步执行将命令排队在线程池中并返回Future：

```java
chain.get("async", ctx -> ctx.render(
    new HystrixAsyncHttpCommand(eugenGithubProfileUri, timeout)
        .queue()
        .get()));
```

HystrixAsyncHttpCommand如下所示：

```java
public class HystrixAsyncHttpCommand extends HystrixCommand<String> {

    //...

    @Override
    protected String run() throws Exception {
        return EntityUtils.toString(HttpClientBuilder.create()
                .setDefaultRequestConfig(requestConfig)
                .setDefaultHeaders(Collections.singleton(
                        new BasicHeader("User-Agent", "Tuyucheng Blocking HttpClient")))
                .build().execute(new HttpGet(uri)).getEntity());
    }

    @Override
    protected String getFallback() {
        return "tuyucheng's async fallback profile";
    }
}
```

这里我们使用阻塞[HttpClient](https://hc.apache.org/httpcomponents-core-5.2.x/examples.html)而不是非阻塞HttpClient，因为我们希望Hystrix控制实际命令的执行超时，这样我们在从Future获取响应时就不需要自己处理它；这也允许Hystrix回退或缓存我们的请求。

异步执行也产生预期的结果：

```java
@Test
public void whenFetchAsync_thenGotEugenProfile() {
    assertThat(appUnderTest.getHttpClient().getText("async"), containsString("www.tuyucheng.com"));
}
```

### 3.3 同步命令执行

同步执行直接在当前线程中执行命令：

```java
chain.get("sync", ctx -> ctx.render(
    new HystrixSyncHttpCommand(eugenGithubProfileUri, timeout).execute()));
```

HystrixSyncHttpCommand的实现与HystrixAsyncHttpCommand几乎相同，只是我们给它一个不同的回退结果。当不回退时，它的行为与响应式和异步执行相同：

```java
@Test
public void whenFetchSync_thenGotEugenProfile() {
    assertThat(appUnderTest.getHttpClient().getText("sync"), containsString("www.tuyucheng.com"));
}
```

## 4. 指标

通过将[Guice模块](https://www.baeldung.com/ratpack-google-guice)-HystrixModule注册到Ratpack注册表中，我们可以流式传输请求范围指标并通过GET端点公开事件流：

```java
serverSpec.registry(
    Guice.registry(spec -> spec.module(new HystrixModule().sse())))
    .handlers(c -> c.get("hystrix", new HystrixMetricsEventStreamHandler()));
```

HystrixMetricsEventStreamHandler帮助以文本/事件流格式传输Hystrix指标，以便我们可以在[Hystrix Dashboard](https://github.com/Netflix-Skunkworks/hystrix-dashboard)中监控这些指标。

我们可以设置一个[独立的Hystrix仪表板](https://github.com/kennedyoliveira/standalone-hystrix-dashboard)，并将我们的Hystrix事件流添加到监视列表中，以查看我们的Ratpack应用程序的执行情况：

![](/assets/images/2025/webmodules/ratpackhystrix01.png)

对我们的Ratpack应用程序发出几次请求后，我们可以在仪表板中看到Hystrix相关的命令。

### 4.1 内部原理

在HystrixModule中，[Hystrix并发策略](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/strategy/concurrency/HystrixConcurrencyStrategy.html)通过[HystrixPlugin](https://github.com/Netflix/Hystrix/wiki/Plugins)注册到Hystrix，以便使用Ratpack注册表管理请求上下文，这样就无需在每个请求开始前初始化Hystrix请求上下文。

```java
public class HystrixModule extends ConfigurableModule<HystrixModule.Config> {

    //...

    @Override
    protected void configure() {
        try {
            HystrixPlugins.getInstance().registerConcurrencyStrategy(
                    new HystrixRegistryBackedConcurrencyStrategy());
        } catch (IllegalStateException e) {
            //...
        }
    }

    //...
}
```

## 5. 总结

在这篇简短的文章中，我们展示了如何将Hystrix集成到Ratpack中，以及如何将Ratpack应用程序的指标推送到Hystrix仪表板，以便更好地查看应用程序的性能。