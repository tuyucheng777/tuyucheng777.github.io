---
layout: post
title:  将zipWhen()与Mono结合使用
category: spring-reactive
copyright: spring-reactive
excerpt: Spring Reactive
---

## 1. 概述

在本教程中，我们将探索如何使用[zipWhen()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#zipWhen-java.util.function.Function-)以协调的方式组合两个或多个[Mono](https://www.baeldung.com/java-reactor-flux-vs-mono)流的结果。我们将从快速概述开始，接下来，我们将设置一个涉及用户数据存储和电子邮件的简单示例。我们将展示zipWhen()如何使我们能够在需要同时收集和处理来自各种来源的数据的情况下协调多个异步操作。

## 2. 什么是zipWhen()？

在使用Mono的[响应式编程](https://www.baeldung.com/java-reactive-systems)中，zipWhen()是一个运算符，它允许我们以协调的方式组合两个或多个Mono流的结果。**当我们要同时执行多个异步操作，并且需要将它们的结果组合成一个输出时，通常会使用它**。

我们从两个或多个代表异步操作的Mono流开始，这些Mono可以发出不同类型的数据，并且它们可能相互依赖，也可能不相互依赖。

然后我们使用zipWhen()进行协调，我们将zipWhen()运算符应用于其中一个Mono。此运算符等待第一个Mono发出一个值，然后使用该值触发其他Mono的执行。zipWhen()的结果是一个新的Mono，它将所有Mono的结果组合成一个数据结构，通常是[Tuple](https://www.baeldung.com/java-tuples)或我们定义的对象。

最后，我们可以指定如何组合Mono的结果。我们可以使用组合的值来创建新对象、执行计算或构建有意义的响应。

## 3. 示例设置

让我们设置一个由3个简化服务组成的简单示例：UserService、EmailService和DataBaseService。它们每个都以不同类型的Mono形式生成数据，我们希望将所有数据组合成一个响应并将其返回给调用客户端。让我们首先设置适当的[POM依赖](https://www.baeldung.com/maven)。

### 3.1 依赖

我们需要[spring-boot-starter-webflux](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-webflux/3.1.3)和[react-test](https://mvnrepository.com/artifact/io.projectreactor/reactor-test/3.5.10)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 3.2 设置UserService

我们首先介绍一下用户服务：

```java
public class UserService {
    public Mono<User> getUser(String userId) {
        return Mono.just(new User(userId, "John Stewart"));
    }
}
```

这里，UserService提供了一种根据给定的userId检索用户数据的方法。它返回一个代表用户信息的Mono<User\>。

### 3.3 设置EmailService

接下来，让我们添加EmailService：

```java
public class EmailService {
    private final UserService userService;

    public EmailService(UserService userService) {
        this.userService = userService;
    }

    public Mono<Boolean> sendEmail(String userId) {
        return userService.getUser(userId)
                .flatMap(user -> {
                    System.out.println("Sending email to: " + user.getEmail());
                    return Mono.just(true);
                })
                .defaultIfEmpty(false);
    }
}
```

顾名思义，EmailService负责向用户发送电子邮件。**重要的是，它依赖于UserService来获取用户详细信息，然后根据检索到的信息发送电子邮件**。sendEmail()方法返回一个Mono<Boolean\>，指示电子邮件是否发送成功。

### 3.4 设置DatabaseService

```java
public class DatabaseService {
    private Map<String, User> dataStore = new ConcurrentHashMap<>();

    public Mono<Boolean> saveUserData(User user) {
        return Mono.create(sink -> {
            try {
                dataStore.put(user.getId(), user);
                sink.success(true);
            } catch (Exception e) {
                sink.success(false);
            }
        });
    }
}
```

DatabaseService负责将用户数据持久化到数据库，为简单起见，我们在这里使用并发Map来表示数据存储。

它提供了一个saveUserData()方法，该方法获取用户信息并返回一个Mono<Boolean\>来表示数据库操作的成功或失败。

## 4. zipWhen()的实际操作

现在我们已经定义了所有服务，让我们定义一个控制器方法，将来自所有3个服务的Mono流组合成一个Mono<ResponseEntity<String>\>类型的响应。我们将展示如何使用zipWhen()运算符来协调各种异步操作并将它们全部转换为调用客户端的单个响应，首先定义GET方法：

```java
@GetMapping("/example/{userId}")
public Mono<ResponseEntity<String>> combineAllDataFor(@PathVariable String userId) {
    Mono<User> userMono = userService.getUser(userId);
    Mono<Boolean> emailSentMono = emailService.sendEmail(userId)
            .subscribeOn(Schedulers.parallel());
    Mono<String> databaseResultMono = userMono.flatMap(user -> databaseService.saveUserData(user)
            .map(Object::toString));

    return userMono.zipWhen(user -> emailSentMono, (t1, t2) -> Tuples.of(t1, t2))
            .zipWhen(tuple -> databaseResultMono, (tuple, databaseResult) -> {
                User user = tuple.getT1();
                Boolean emailSent = tuple.getT2();
                return ResponseEntity.ok()
                        .body("Response: " + user + ", Email Sent: " + emailSent + ", Database Result: " + databaseResult);
            });
}
```

当客户端调用GET /example/{userId}端点时，userService将调用CombineAllData()方法，通过调用userService.getUser(userId)根据提供的userId检索有关用户的信息，此结果存储在此处名为userMono的Mono<User\>中。

接下来，它向同一用户发送电子邮件。但是，在发送电子邮件之前，它会检查该用户是否存在。电子邮件发送操作的结果(成功或失败)由Mono<Boolean\>类型的emailSentMono表示，此操作并行执行以节省时间。它使用databaseService.saveUserData(user)将用户数据(在步骤1中检索)保存到数据库，此操作的结果(成功或失败)将转换为字符串并存储在Mono<String\>中。

重要的是，它使用zipWhen()运算符来组合前面步骤的结果。**第一个zipWhen()将用户数据userMono和emailSentMono中的电子邮件发送状态组合成一个元组。第二个zipWhen()将前一个元组和dataBaseResultMono中的数据库结果组合起来以构造最终响应**。在第二个zipWhen()中，它使用组合数据构造响应消息。

该消息包括用户信息、电子邮件是否发送成功以及数据库操作的结果。本质上，此方法协调特定用户的用户数据检索、电子邮件发送和数据库操作，并将结果组合成有意义的响应，确保所有操作高效且并发地进行。

## 5. 测试

现在，让我们测试一下系统，并验证是否返回了结合了3种不同类型的响应流的正确响应：

```java
@Test
public void givenUserId_whenCombineAllData_thenReturnsMonoWithCombinedData() {
    UserService userService = Mockito.mock(UserService.class);
    EmailService emailService = Mockito.mock(EmailService.class);
    DatabaseService databaseService = Mockito.mock(DatabaseService.class);

    String userId = "123";
    User user = new User(userId, "John Doe");

    Mockito.when(userService.getUser(userId))
            .thenReturn(Mono.just(user));
    Mockito.when(emailService.sendEmail(userId))
            .thenReturn(Mono.just(true));
    Mockito.when(databaseService.saveUserData(user))
            .thenReturn(Mono.just(true));

    UserController userController = new UserController(userService, emailService, databaseService);

    Mono<ResponseEntity<String>> responseMono = userController.combineAllDataFor(userId);

    StepVerifier.create(responseMono)
            .expectNextMatches(responseEntity -> responseEntity.getStatusCode() == HttpStatus.OK && responseEntity.getBody()
                    .equals("Response: " + user + ", Email Sent: true, Database Result: " + true))
            .verifyComplete();
}
```

我们使用[StepVerifier](https://www.baeldung.com/reactive-streams-step-verifier-test-publisher)来验证响应实体是否具有预期的200 OK状态码，以及使用zipWhen()组合不同Mono结果的主体。

## 6. 总结

在本教程中，我们快速了解了在响应式编程中如何使用zipWhen()和Mono。我们使用了用户数据收集、电子邮件和存储组件的示例，所有这些组件都提供了不同类型的Mono。此示例演示了如何使用zipWhen()有效地处理数据依赖关系并在响应式Spring WebFlux应用程序中协调异步操作。