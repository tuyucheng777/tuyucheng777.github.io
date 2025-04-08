---
layout: post
title:  在Spring Boot中将CompletableFuture与Feign Client结合使用
category: springcloud
copyright: springcloud
excerpt: Spring Cloud OpenFeign
---

## 1. 简介

在使用分布式系统时，调用外部Web依赖并保持低延迟是一项关键任务。

在本教程中，我们将使用[OpenFeign](https://www.baeldung.com/spring-cloud-openfeign)和[CompletableFuture](https://www.baeldung.com/java-completablefuture)并行化多个HTTP请求、处理错误以及设置网络和线程超时。

## 2. 设置演示应用程序

为了说明并行请求的用法，**我们将创建一项功能，允许客户在网站上购买商品**。首先，该服务发出一个请求，根据客户居住的国家/地区获取可用的付款方式。其次，它发出一个请求，向客户生成有关购买的报告。购买报告不包含有关付款方式的信息。

因此，让我们首先添加依赖以与[spring-cloud-starter-openfeign](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-openfeign)配合使用：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

## 3. 创建外部依赖客户端

现在，**让我们使用@FeignClient注解创建两个指向localhost:8083的客户端**：

```java
@FeignClient(name = "paymentMethodClient", url = "http://localhost:8083")
public interface PaymentMethodClient {

    @RequestMapping(method = RequestMethod.GET, value = "/payment_methods")
    String getAvailablePaymentMethods(@RequestParam(name = "site_id") String siteId);
}
```

我们的第一个客户端名称是paymentMethodClient，它调用GET /payment_methods来获取可用的付款方式，并使用代表客户所在国家/地区的site_id请求参数。

让我们看看我们的第二个客户端：

```java
@FeignClient(name = "reportClient", url = "http://localhost:8083")
public interface ReportClient {

    @RequestMapping(method = RequestMethod.POST, value = "/reports")
    void sendReport(@RequestBody String reportRequest);
}
```

我们将其命名为reportClient，它调用POST /reports来生成购买报告。

## 4. 创建并行请求执行器

按顺序调用两个客户端就足以完成演示应用程序的要求，在这种情况下，**此API的总响应时间至少是两个请求响应时间的总和**。

值得注意的是，报告不包含有关付款方式的信息，因此这两个请求是独立的。因此，**我们可以并行化工作，以将API的总响应时间缩短至与最慢请求的响应时间大致相同**。

在接下来的部分中，我们将看到如何创建HTTP调用的并行执行器并处理外部错误。

### 4.1 创建并行执行器

因此，**让我们使用CompletableFuture来创建并行处理两个请求的服务**：

```java
@Service
public class PurchaseService {

    private final PaymentMethodClient paymentMethodClient;
    private final ReportClient reportClient;

    // all-arg constructor

    public String executePurchase(String siteId) throws ExecutionException, InterruptedException {
        CompletableFuture<String> paymentMethodsFuture = CompletableFuture.supplyAsync(() ->
                paymentMethodClient.getAvailablePaymentMethods(siteId));
        CompletableFuture.runAsync(() -> reportClient.sendReport("Purchase Order Report"));

        return String.format("Purchase executed with payment method %s", paymentMethodsFuture.get());
    }
}
```

executePurchase()方法首先使用supplyAsync()发布一个并行任务以获取可用的付款方式。然后，我们提交另一个并行任务以使用runAsync()生成报告。最后，我们使用get()检索付款方式结果并返回完整结果。

为这两项任务选择[supplyAsync()和runAsync()](https://www.baeldung.com/java-completablefuture-runasync-supplyasync)是因为这两种方法的性质不同，supplyAsync()方法返回GET调用的结果。另一方面，runAsync()不返回任何内容，因此它更适合生成报告。

另一个区别是，runAsync()会在我们调用代码时立即启动一个新线程，而无需线程池进行任何任务调度。相反，supplyAsync()任务可能会被调度或延迟，具体取决于线程池是否调度了其他任务。

为了验证我们的代码，让我们使用[WireMock](https://www.baeldung.com/introduction-to-wiremock)进行集成测试：

```java
@BeforeEach
public void startWireMockServer() {
    wireMockServer = new WireMockServer(8083);
    configureFor("localhost", 8083);
    wireMockServer.start();

    stubFor(post(urlEqualTo("/reports"))
            .willReturn(aResponse().withStatus(HttpStatus.OK.value())));
}

@AfterEach
public void stopWireMockServer() {
    wireMockServer.stop();
}

@Test
void givenRestCalls_whenBothReturnsOk_thenReturnCorrectResult() throws ExecutionException, InterruptedException {
    stubFor(get(urlEqualTo("/payment_methods?site_id=BR"))
            .willReturn(aResponse().withStatus(HttpStatus.OK.value()).withBody("credit_card")));

    String result = purchaseService.executePurchase("BR");

    assertNotNull(result);
    assertEquals("Purchase executed with payment method credit_card", result);
}
```

在上面的测试中，我们首先配置一个WireMockServer在localhost:8083启动，并在完成后使用@BeforeEach和@AfterEach注解关闭。

然后，在测试场景方法中，我们使用两个存根，在调用两个Feign客户端时，它们均以200 HTTP状态响应。最后，我们使用assertEquals()从并行执行器断言正确的结果。

### 4.2 使用exceptionally()处理外部API错误

如果GET /payment_methods请求失败并显示404 HTTP状态，表明该国家/地区没有可用的付款方式，该怎么办？**在这种情况下，执行某些操作很有用，例如返回默认值**。

为了处理[CompletableFuture中的错误](https://www.baeldung.com/java-exceptions-completablefuture)，让我们将以下exceptionally()块添加到paymentMethodsFuture中：

```java
CompletableFuture <String> paymentMethodsFuture = CompletableFuture.supplyAsync(() -> paymentMethodClient.getAvailablePaymentMethods(siteId))
    .exceptionally(ex -> {
        if (ex.getCause() instanceof FeignException && ((FeignException) ex.getCause()).status() == 404) {
            return "cash";
        });
```

现在，如果我们收到404错误，我们将返回名为cash的默认付款方式：

```java
@Test
void givenRestCalls_whenPurchaseReturns404_thenReturnDefault() throws ExecutionException, InterruptedException {
    stubFor(get(urlEqualTo("/payment_methods?site_id=BR"))
        .willReturn(aResponse().withStatus(HttpStatus.NOT_FOUND.value())));

    String result = purchaseService.executePurchase("BR");

    assertNotNull(result);
    assertEquals("Purchase executed with payment method cash", result);
}
```

## 5. 为并行任务和网络请求添加超时

调用外部依赖时，我们无法确定请求需要多长时间才能运行。因此，**如果请求耗时过长，在某个时候，我们应该放弃该请求**。考虑到这一点，我们可以添加两种类型：FeignClient和CompletableFuture超时。

### 5.1 为Feign客户端添加网络超时

**这种超时适用于通过网络进行的单个请求。因此，它会在网络级别切断与单个请求的外部依赖关系的连接**。

我们可以使用Spring Boot自动配置为FeignClient配置超时：

```yaml
feign.client.config.paymentMethodClient.readTimeout: 200
feign.client.config.paymentMethodClient.connectTimeout: 100
```

在上面的application.yaml文件中，我们为PaymentMethodClient设置了读取和连接超时，数值以毫秒为单位。

连接超时指示Feign客户端在达到阈值后切断TCP握手连接尝试。类似地，当连接正确建立但协议无法从套接字读取数据时，读取超时会中断请求。

然后，我们可以在并行执行器中的exceptionally()块内处理该类型的错误：

```java
if (ex.getCause() instanceof RetryableException) {
    // handle TCP timeout
    throw new RuntimeException("TCP call network timeout!");
}
```

为了验证正确的行为，我们可以添加另一个测试场景：

```java
@Test
void givenRestCalls_whenPurchaseRequestWebTimeout_thenReturnDefault() {
    stubFor(get(urlEqualTo("/payment_methods?site_id=BR"))
        .willReturn(aResponse().withFixedDelay(250)));

    Throwable error = assertThrows(ExecutionException.class, () -> purchaseService.executePurchase("BR"));

    assertEquals("java.lang.RuntimeException: REST call network timeout!", error.getMessage());
}
```

在这里，我们使用了250毫秒的withFixedDelay()方法来模拟TCP超时。

### 5.2 添加线程超时

另一方面，**线程超时会停止整个CompletableFuture内容，而不仅仅是单个请求尝试**。例如，对于[Feign客户端重试](https://www.baeldung.com/feign-retry)，在评估超时阈值时，原始请求和重试尝试的时间也会计入。

为了配置线程超时，我们可以稍微修改我们的付款方法CompletableFuture：

```java
CompletableFuture<String> paymentMethodsFuture = CompletableFuture.supplyAsync(() -> paymentMethodClient.getAvailablePaymentMethods(siteId))
    .orTimeout(400, TimeUnit.MILLISECONDS)
    .exceptionally(ex -> {
         // exception handlers
    });
```

然后，我们可以在exceptionally()块内处理威胁超时错误：

```java
if (ex instanceof TimeoutException) {
    // handle thread timeout
    throw new RuntimeException("Thread timeout!", ex);
}
```

因此，我们可以验证它是否正常工作：

```java
@Test
void givenRestCalls_whenPurchaseCompletableFutureTimeout_thenThrowNewException() {
    stubFor(get(urlEqualTo("/payment_methods?site_id=BR"))
        .willReturn(aResponse().withFixedDelay(450)));

    Throwable error = assertThrows(ExecutionException.class, () -> purchaseService.executePurchase("BR"));

    assertEquals("java.lang.RuntimeException: Thread timeout!", error.getMessage());
}
```

我们为/payments_method添加了更长的延迟，以便它可以通过网络超时阈值，但因线程超时而失败。

## 6. 总结

在本文中，我们学习了如何使用CompletableFuture和FeignClient并行执行两个外部依赖请求。

我们还了解了如何添加网络和线程超时以在时间阈值之后中断程序执行。

最后，我们使用CompletableFuture.exceptionally()优雅地处理了404 API和超时错误。