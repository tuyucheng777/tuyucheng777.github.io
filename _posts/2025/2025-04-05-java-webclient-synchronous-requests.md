---
layout: post
title:  使用WebClient执行同步请求
category: springreactive
copyright: springreactive
excerpt: Spring Reactive
---

## 1. 简介

在本教程中，我们将学习如何使用[WebClient](https://www.baeldung.com/tag/webclient)执行同步请求。

随着[响应式编程](https://www.baeldung.com/cs/reactive-programming)变得越来越普及，我们将研究此类阻塞请求仍然适当且必要的场景。

## 2. Spring中的HTTP客户端库概述

让我们首先简要回顾一下当前可用的客户端库，看看我们的WebClient适合哪里。

[RestTemplate](https://www.baeldung.com/rest-template)在Spring 3.0中推出后，因其用于HTTP请求的简单模板方法API而广受欢迎。然而，其同步特性和许多重载方法导致高流量应用程序中的复杂性和性能瓶颈。

在[Spring 5.0](https://www.baeldung.com/spring-5)中，引入了WebClient作为非阻塞请求的更高效、响应式替代方案。虽然它是[响应堆栈Web框架](https://www.baeldung.com/spring-webflux)的一部分，但它支持用于同步和异步通信的流式API。

在Spring 6.1 中，[RestClient](https://www.baeldung.com/spring-boot-restclient)提供了另一种执行REST调用的选项。它将WebClient的流式API与RestTemplate的基础架构(包括消息转换器、请求工厂和拦截器)相结合。

**RestClient针对同步请求进行了优化，但如果我们的应用程序还需要异步或流式传输功能，则WebClient会更好**。使用WebClient进行阻塞和非阻塞API调用，我们可以保持代码库的一致性，并避免混合不同的客户端库。

## 3. 阻塞与非阻塞API调用

在讨论各种HTTP客户端时，我们使用了同步和异步、阻塞和非阻塞等术语。这些术语与上下文相关，有时可能代表同一概念的不同名称。

**在方法调用方面，WebClient根据其发送和接收HTTP请求和响应的方式支持同步和异步交互**。如果它等待前一个请求完成后再继续执行后续请求，则它以阻塞方式执行此操作，并且结果将同步返回。

另一方面，我们可以通过执行立即返回的非阻塞调用来实现[异步](https://www.baeldung.com/cs/async-vs-multi-threading#what-is-asynchronous-programming)交互。在等待另一个系统的响应时，其他处理可以继续，一旦准备就绪，结果就会异步提供。

## 4. 何时使用同步请求

如上所述，WebClient是Spring Webflux框架的一部分，默认情况下，该框架中的所有内容都是响应式的。但是，该库提供异步和同步操作支持，使其适用于响应式和Servlet堆栈Web应用程序。

**当需要立即反馈时(例如在测试或原型设计期间)，以阻塞方式使用WebClient是合适的**，这种方法使我们在考虑性能优化之前可以专注于功能。

许多现有应用程序仍在使用RestTemplate等阻塞客户端。由于RestTemplate从Spring 5.0开始处于维护模式，因此重构遗留代码库将需要更新依赖，并可能需要过渡到非阻塞架构。在这种情况下，我们可以暂时以阻塞方式使用WebClient。

即使在新项目中，某些应用程序部分也可能设计为同步工作流。这可以包括对各种外部系统的顺序API调用等场景，其中一个调用的结果对于下一个调用是必需的。WebClient可以处理阻塞和非阻塞调用，而不必使用不同的客户端。

我们稍后会看到，同步和异步执行之间的切换相对简单。**只要有可能，我们就应该避免使用阻塞调用，尤其是在使用响应式堆栈时**。

## 5. 使用WebClient进行同步API调用

发送HTTP请求时，WebClient会从[Reactor Core](https://www.baeldung.com/reactor-core)库中返回两种响应数据类型之一-[Mono或Flux](https://www.baeldung.com/java-reactor-flux-vs-mono)。这些返回类型表示数据流，其中Mono对应于单个值或空结果，而Flux指的是零个或多个值的流。拥有异步和非阻塞API可让调用者决定何时以及如何订阅，从而保持代码的响应式。

**但是，如果我们想模拟同步行为，我们可以调用可用的block()方法**，它将阻塞当前操作以获取结果。

更准确地说，block()方法触发对响应流的新订阅，从而启动从源到消费者的数据流。在内部，它使用[CountDownLatch](https://www.baeldung.com/java-countdown-latch)等待流完成，这会暂停当前线程直到操作完成，即直到Mono或Flux发出结果。**block()方法将非阻塞操作转换为传统的阻塞操作，导致调用线程等待结果**。

## 6. 实例

让我们看看实际效果，想象一个位于客户端应用程序和两个后端应用程序(客户和计费系统)之间的简单[API网关](https://www.baeldung.com/cs/api-gateway-vs-reverse-proxy)应用程序。第一个保存客户信息，而第二个提供账单详细信息。不同的客户端通过[北向](https://www.baeldung.com/cs/network-traffic-north-south-east-west#north-south-and-east-west-traffic)API与我们的API网关交互，北向API是向客户端公开的接口，用于检索客户信息，包括他们的账单详细信息：

```java
@GetMapping("/{id}")
CustomerInfo getCustomerInfo(@PathVariable("id") Long customerId) {
    return customerInfoService.getCustomerInfo(customerId);
}
```

模型类如下所示：

```java
public class CustomerInfo {
    private Long customerId;
    private String customerName;
    private Double balance;

    // standard getters and setters
}
```

API网关通过提供与客户和计费应用程序进行内部通信的单一端点简化了流程，然后，它会汇总来自两个系统的数据。

考虑一下我们在整个系统中使用同步API的场景。但是，我们最近升级了客户和计费系统以处理异步和非阻塞操作。让我们看看这两个南向API现在是什么样子。

客户API：

```java
@GetMapping("/{id}")
Mono<Customer> getCustomer(@PathVariable("id") Long customerId) throws InterruptedException {
    TimeUnit.SECONDS.sleep(SLEEP_DURATION.getSeconds());
    return Mono.just(customerService.getBy(customerId));
}
```

计费API：

```java
@GetMapping("/{id}")
Mono<Billing> getBilling(@PathVariable("id") Long customerId) throws InterruptedException {
    TimeUnit.SECONDS.sleep(SLEEP_DURATION.getSeconds());
    return Mono.just(billingService.getBy(customerId));
}
```

在实际情况下，这些API将是单独组件的一部分。但是，为了简单起见，我们已将它们组织到代码中的不同包中。此外，为了进行测试，我们引入了睡眠来模拟网络延迟：

```java
public static final Duration SLEEP_DURATION = Duration.ofSeconds(2);
```

与两个后端系统不同，我们的API网关应用程序必须公开同步、阻塞API，以避免破坏客户端契约。因此，那里没有任何变化。

业务逻辑位于CustomerInfoService中。首先，我们将使用WebClient从客户系统中检索数据：

```java
Customer customer = webClient.get()
    .uri(uriBuilder -> uriBuilder.path(CustomerController.PATH_CUSTOMER)
        .pathSegment(String.valueOf(customerId))
        .build())
    .retrieve()
    .onStatus(status -> status.is5xxServerError() || status.is4xxClientError(), response -> response.bodyToMono(String.class)
        .map(ApiGatewayException::new))
    .bodyToMono(Customer.class)
    .block();
```

接下来是计费系统：

```java
Billing billing = webClient.get()
    .uri(uriBuilder -> uriBuilder.path(BillingController.PATH_BILLING)
        .pathSegment(String.valueOf(customerId))
        .build())
    .retrieve()
    .onStatus(status -> status.is5xxServerError() || status.is4xxClientError(), response -> response.bodyToMono(String.class)
        .map(ApiGatewayException::new))
    .bodyToMono(Billing.class)
    .block();
```

最后，使用两个组件的响应，我们将构建一个响应：

```java
new CustomerInfo(customer.getId(), customer.getName(), billing.getBalance());
```

如果某个API调用失败，则onStatus()方法中定义的错误处理会将HTTP错误状态映射到ApiGatewayException。在这里，我们使用[传统方法](https://www.baeldung.com/spring-webflux-difference-exception-mono#embracing-reactivity-monoerror)，而不是通过Mono.error()方法采用响应式替代方法。由于我们的客户期望使用同步API，因此我们抛出了会传播给调用者的异常。

尽管客户和计费系统具有异步特性，WebClient的block()方法使我们能够从两个来源聚合数据并向客户端透明地返回组合结果。

### 6.1 优化多个API调用

此外，由于我们要对不同的系统进行两次连续调用，因此我们可以通过避免单独阻塞每个响应来优化流程，我们可以执行以下操作：

```java
private CustomerInfo getCustomerInfoBlockCombined(Long customerId) {
    Mono<Customer> customerMono = webClient.get()
            .uri(uriBuilder -> uriBuilder.path(CustomerController.PATH_CUSTOMER)
                    .pathSegment(String.valueOf(customerId))
                    .build())
            .retrieve()
            .onStatus(status -> status.is5xxServerError() || status.is4xxClientError(), response -> response.bodyToMono(String.class)
                    .map(ApiGatewayException::new))
            .bodyToMono(Customer.class);

    Mono<Billing> billingMono = webClient.get()
            .uri(uriBuilder -> uriBuilder.path(BillingController.PATH_BILLING)
                    .pathSegment(String.valueOf(customerId))
                    .build())
            .retrieve()
            .onStatus(status -> status.is5xxServerError() || status.is4xxClientError(), response -> response.bodyToMono(String.class)
                    .map(ApiGatewayException::new))
            .bodyToMono(Billing.class);

    return Mono.zip(customerMono, billingMono, (customer, billing) -> new CustomerInfo(customer.getId(), customer.getName(), billing.getBalance()))
            .block();
}
```

zip()是一种将多个Mono实例组合成单个Mono的方法，当所有给定的Mono都生成了它们的值时，新的Mono就完成了，然后根据指定的函数对这些值进行聚合-在我们的例子中，创建了一个CustomerInfo对象。这种方法更高效，因为它允许我们同时等待两个服务的组合结果。

为了验证我们是否提高了性能，让我们在两种情况下运行测试：

```java
@Autowired
private WebTestClient testClient;

@Test
void givenApiGatewayClient_whenBlockingCall_thenResponseReceivedWithinDefinedTimeout() {
    Long customerId = 10L;
    assertTimeout(Duration.ofSeconds(CustomerController.SLEEP_DURATION.getSeconds() + BillingController.SLEEP_DURATION.getSeconds()), () -> {
        testClient.get()
                .uri(uriBuilder -> uriBuilder.path(ApiGatewayController.PATH_CUSTOMER_INFO)
                        .pathSegment(String.valueOf(customerId))
                        .build())
                .exchange()
                .expectStatus()
                .isOk();
    });
}
```

最初，测试失败了。但是，在切换到等待合并结果后，测试在客户和计费系统调用的总持续时间内完成，**这表明我们通过聚合两个服务的响应提高了性能**。即使我们使用阻塞同步方法，我们仍然可以遵循最佳实践来优化性能，这有助于确保系统保持高效和可靠。

## 7. 总结

在本教程中，我们演示了如何使用WebClient管理同步通信，WebClient是一种专为响应式编程设计但能够进行阻塞调用的工具。

总而言之，我们讨论了使用WebClient相对于其他库(如RestClient)的优势，尤其是在响应式堆栈中，以保持一致性并避免混合不同的客户端库。最后，我们探索了通过聚合来自多个服务的响应而不阻塞每个调用来优化性能。