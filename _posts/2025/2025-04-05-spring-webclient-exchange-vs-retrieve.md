---
layout: post
title:  Spring WebClient exchange()与retrieve()
category: springreactive
copyright: springreactive
excerpt: Spring Reactive
---

## 1. 概述

[WebClient](https://www.baeldung.com/spring-5-webclient)是一个简化执行HTTP请求过程的接口，与[RestTemplate](https://www.baeldung.com/spring-webclient-resttemplate)不同，它是一个响应式、非阻塞的客户端，可以使用和操作HTTP响应。虽然它被设计为非阻塞的，但它也可以在阻塞场景中使用。

在本教程中，我们将深入研究WebClient接口中的关键方法，包括tries()、exchangeToMono()和exchangeToFlux()。此外，我们将探讨这些方法之间的差异和相似之处，并通过示例以展示不同的用例。此外，我们将使用[JSONPlaceholder](https://jsonplaceholder.typicode.com/) API来获取用户数据。

## 2. 示例设置

首先，让我们启动一个Spring Boot应用程序并将[spring-boot-starter-webflux](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-webflux)依赖添加到pom.xml：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
    <version>3.2.4</version>
</dependency>
```

**此依赖提供了WebClient接口，使我们能够执行HTTP请求**。

另外，让我们看一下对https://jsonplaceholder.typicode.com/users/1的请求的示例GET响应：

```json
{
    "id": 1,
    "name": "Leanne Graham",
    // ...
}
```

此外，让我们创建一个名为User的[POJO类](https://www.baeldung.com/java-pojo-class)：

```java
class User {

    private int id;
    private String name;

    // standard constructor,getter, and setter
}
```

JSONPlaceholder API的JSON响应将被反序列化并映射到User类的实例。

最后，让我们使用基本URL创建一个WebClient实例：

```java
WebClient client = WebClient.create("https://jsonplaceholder.typicode.com/users");
```

在这里，我们定义HTTP请求的基本URL。

## 3. exchange()方法

**exchange()方法直接返回ClientResponse，从而提供对HTTP状态码、标头和响应主体的访问**。简而言之，ClientResponse表示由WebClient返回的HTTP响应。

**但是，自Spring版本5.3以来，此方法已被弃用，并已根据我们发出的内容由exchangeToMono()或exchangeToFlux()方法取代**，这两种方法允许我们根据响应状态解码响应。

### 3.1 发射Mono

让我们看一个使用exchangeToMono()发出[Mono](https://www.baeldung.com/java-reactor-flux-vs-mono#what-is-mono)的示例：

```java
@GetMapping("/user/exchange-mono/{id}")
Mono<User> retrieveUsersWithExchangeAndError(@PathVariable int id) {
    return client.get()
            .uri("/{id}", id)
            .exchangeToMono(res -> {
                if (res.statusCode().is2xxSuccessful()) {
                    return res.bodyToMono(User.class);
                } else if (res.statusCode().is4xxClientError()) {
                    return Mono.error(new RuntimeException("Client Error: can't fetch user"));
                } else if (res.statusCode().is5xxServerError()) {
                    return Mono.error(new RuntimeException("Server Error: can't fetch user"));
                } else {
                    return res.createError();
                }
            });
}
```

在上面的代码中，我们检索用户并根据HTTP状态码解码响应。

### 3.2 发射Flux

此外，让我们使用exchangeToFlux()来获取用户集合：

```java
@GetMapping("/user-exchange-flux")
Flux<User> retrieveUsersWithExchange() {
    return client.get()
            .exchangeToFlux(res -> {
                if (res.statusCode().is2xxSuccessful()) {
                    return res.bodyToFlux(User.class);
                } else {
                    return Flux.error(new RuntimeException("Error while fetching users"));
                }
            });
}
```

在这里，我们使用exchangeToFlux()方法将响应主体映射到User对象的[Flux](https://www.baeldung.com/java-reactor-flux-vs-mono#what-is-flux)，并在请求失败时返回自定义错误消息。

### 3.3 直接获取响应主体

值得注意的是，exchangeToMono()或exchangeToFlux()可以在不指定响应状态码的情况下使用：

```java
@GetMapping("/user-exchange")
Flux<User> retrieveAllUserWithExchange(@PathVariable int id) {
    return client.get().exchangeToFlux(res -> res.bodyToFlux(User.class))
        .onErrorResume(Flux::error);
}
```

在这里，我们检索用户而不指定状态码。

### 3.4 修改响应主体

此外，让我们看一个修改响应主体的例子：

```java
@GetMapping("/user/exchange-alter/{id}")
Mono<User> retrieveOneUserWithExchange(@PathVariable int id) {
    return client.get()
            .uri("/{id}", id)
            .exchangeToMono(res -> res.bodyToMono(User.class))
            .map(user -> {
                user.setName(user.getName().toUpperCase());
                user.setId(user.getId() + 100);
                return user;
            });
}
```

在上面的代码中，将响应体映射到POJO类后，我们通过将id加100并将name大写来改变响应体。

值得注意的是，我们还可以使用retrieve()方法来改变响应主体。

### 3.5 提取响应头

另外，我们可以提取响应头：

```java
@GetMapping("/user/exchange-header/{id}")
Mono<User> retrieveUsersWithExchangeAndHeader(@PathVariable int id) {
    return client.get()
            .uri("/{id}", id)
            .exchangeToMono(res -> {
                if (res.statusCode().is2xxSuccessful()) {
                    logger.info("Status code: " + res.headers().asHttpHeaders());
                    logger.info("Content-type" + res.headers().contentType());
                    return res.bodyToMono(User.class);
                } else if (res.statusCode().is4xxClientError()) {
                    return Mono.error(new RuntimeException("Client Error: can't fetch user"));
                } else if (res.statusCode().is5xxServerError()) {
                    return Mono.error(new RuntimeException("Server Error: can't fetch user"));
                } else {
                    return res.createError();
                }
            });
}
```

在这里，我们将HTTP标头和内容类型记录到控制台。**与需要返回ResponseEntity才能访问标头和响应码的withdraw()方法不同，exchangeToMono()允许我们直接访问，因为它返回ClientResponse**。

## 4. retrieve()方法

**该retrieve()方法简化了从HTTP请求中提取响应主体的过程，它返回ResponseSpec**，这使我们能够指定如何处理响应主体，而无需访问完整的ClientResponse。

ClientResponse包含响应码、标头和主体。因此，ResponseSpec包含响应主体，但不包含响应码和标头。

### 4.1 发射Mono

以下是检索HTTP响应主体的示例代码：

```java
@GetMapping("/user/{id}")
Mono<User> retrieveOneUser(@PathVariable int id) {
    return client.get()
        .uri("/{id}", id)
        .retrieve()
        .bodyToMono(User.class)
        .onErrorResume(Mono::error);
}
```

在上面的代码中，我们通过使用特定id对/users端点进行HTTP调用，从基本URL检索[JSON](https://www.baeldung.com/java-json)。然后，我们将响应主体映射到User对象。

### 4.2 发射Flux

此外，让我们看一个向/users端点发出GET请求的示例：

```java
@GetMapping("/users")
Flux<User> retrieveAllUsers() {
    return client.get()
        .retrieve()
        .bodyToFlux(User.class)
        .onResumeError(Flux::error);
}
```

这里，当该方法将HTTP响应映射到POJO类时，它会发出一个User对象的Flux。

### 4.3 返回ResponseEntity

如果我们打算使用retrieve()方法访问响应状态和标头，我们可以返回ResponseEntity：

```java
@GetMapping("/user-id/{id}")
Mono<ResponseEntity<User>> retrieveOneUserWithResponseEntity(@PathVariable int id) {
    return client.get()
        .uri("/{id}", id)
        .accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .toEntity(User.class)
        .onErrorResume(Mono::error);
}
```

使用toEntity()方法获得的响应包含HTTP标头、状态码和响应正文。

### 4.4 使用onStatus()处理程序自定义错误

**此外，当出现400或500 HTTP错误时，它默认返回WebClientResponseException错误**。但是，我们可以使用onStatus()处理程序自定义异常以提供自定义错误响应：

```java
@GetMapping("/user-status/{id}")
Mono<User> retrieveOneUserAndHandleErrorBasedOnStatus(@PathVariable int id) {
    return client.get()
        .uri("/{id}", id)
        .retrieve()
        .onStatus(HttpStatusCode::is4xxClientError, response -> Mono.error(new RuntimeException("Client Error: can't fetch user")))
        .onStatus(HttpStatusCode::is5xxServerError, response -> Mono.error(new RuntimeException("Server Error: can't fetch user")))
        .bodyToMono(User.class);
}
```

在这里，我们检查[HTTP状态码](https://www.baeldung.com/cs/http-status-codes)并使用onStatus()处理程序来定义自定义错误响应。

## 5. 性能比较

接下来，让我们编写一个性能测试，使用[Java Microbench Harness(JMH)](https://www.baeldung.com/java-microbenchmark-harness)比较withdraw()和exchangeToFlux()的执行时间。

首先，让我们创建一个名为RetrieveAndExchangeBenchmarkTest的类：

```java
@State(Scope.Benchmark)
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 3, time = 10, timeUnit = TimeUnit.MICROSECONDS)
@Measurement(iterations = 3, time = 10, timeUnit = TimeUnit.MICROSECONDS)
public class RetrieveAndExchangeBenchmarkTest {

    private WebClient client;

    @Setup
    public void setup() {
        this.client = WebClient.create("https://jsonplaceholder.typicode.com/users");
    }
}
```

在这里，我们将基准测试模式设置为AverageTime，这意味着它测量测试执行的平均时间。此外，我们定义了迭代次数和每次迭代的运行时间。

接下来，我们创建一个WebClient实例，并使用@Setup注解使其在每次基准测试之前运行。

让我们编写一个基准测试方法，使用retrieve()方法检索用户集合：

```java
@Benchmark
Flux<User> retrieveManyUserUsingRetrieveMethod() {
    return client.get()
        .retrieve()
        .bodyToFlux(User.class)
        .onErrorResume(Flux::error);;
}
```

最后，让我们定义一个使用exchangeToFlux()方法发出User对象的Flux的方法：

```java
@Benchmark
Flux<User> retrieveManyUserUsingExchangeToFlux() {
    return client.get()
        .exchangeToFlux(res -> res.bodyToFlux(User.class))
        .onErrorResume(Flux::error);
}
```

基准测试结果如下：

```text
Benchmark                             Mode  Cnt   Score    Error  Units
retrieveManyUserUsingExchangeToFlux   avgt   15  ≈ 10⁻⁴            s/op
retrieveManyUserUsingRetrieveMethod   avgt   15  ≈ 10⁻³            s/op
```

两种方法都表现出了高效的性能，但是，在检索用户集合时，exchangeToFlux()比withdraw()方法稍快一些。

## 6. 主要差异和相似之处

withdraw()和exchangeToMono()或exchangeToFlux()均可用于发出HTTP请求并提取HTTP响应。

**由于retrieve()方法返回ResponseSpec，因此它只允许我们使用HTTP主体并发出Mono或Flux**。但是，如果我们想要访问状态码和标头，我们可以将retrieve()方法与ResponseEntity一起使用。此外，它还允许我们使用onStatus()处理程序根据HTTP状态码报告错误。

与retrieve()方法不同，**exchangeToMono()和exchangeToFlux()允许我们使用HTTP响应并直接访问标头和响应码，因为它们返回ClientResponse**。此外，它们提供了对错误处理的更多控制，因为我们可以根据HTTP状态码解码响应。

**值得注意的是，如果只想使用响应主体，建议使用retrieve()方法**。

但是，如果我们需要对响应进行更多的控制，exchangeToMono()或exchangeToFlux()可能是更好的选择。

## 7. 总结

在本文中，我们学习了如何使用retrieve()、exchangeToMono()和exchangeToFlux()方法来处理HTTP响应，并进一步将响应映射到POJO类。此外，我们还比较了retrieve()和exchangeToFlux()方法之间的性能。

在我们只需要使用响应主体而不需要访问状态码或标头的情况下，retrieve()方法非常有用。它通过返回ResponseSpec简化了该过程，从而提供了一种处理响应主体的直接方法。