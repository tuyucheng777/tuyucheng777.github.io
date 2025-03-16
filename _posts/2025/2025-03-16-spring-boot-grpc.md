---
layout: post
title:  Spring Boot中的gRPC简介
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

**[gRPC](https://www.baeldung.com/grpc-introduction)是一个高性能的开源RPC框架，最初由Google开发**。它有助于消除样板代码并连接数据中心内和跨数据中心的多语言服务。该API基于[Protocol Buffers](https://protobuf.dev/)，它提供了一个protoc编译器来为不同的[受支持语言](https://grpc.io/docs/languages/)生成代码。

我们可以将gRPC视为REST、SOAP或GraphQL的替代品，它建立在HTTP/2之上以使用多路复用或[流连接](https://www.baeldung.com/java-grpc-streaming)等功能。

在本教程中，我们将学习如何使用Spring Boot实现gRPC服务提供者和消费者。

## 2. 挑战

首先，我们注意到Spring Boot中没有直接支持gRPC，仅支持Protocol Buffers，这使我们能够实现[基于protobuf的REST服务](https://www.baeldung.com/spring-rest-api-with-protocol-buffers)。因此，我们需要使用第三方库来包含gRPC，或者自己处理一些挑战：

- 依赖于平台的编译器：protoc编译器依赖于平台。因此，如果在构建时生成存根，构建过程会变得更加复杂且容易出错。
- 依赖项：我们需要在Spring Boot应用程序中兼容依赖项。不幸的是，Java的protoc[添加了javax.annotation.Generated注解](https://github.com/grpc/grpc-java/issues/9179)，这迫使我们在编译时添加对旧Java EE注解库的依赖。
- 服务器运行时：gRPC服务提供者需要在服务器中运行，[gRPC for Java](https://github.com/grpc/grpc-java/)项目提供了一个[Shaded Netty](https://github.com/grpc/grpc-java/tree/master/netty/shaded)，我们需要将其包含在Spring Boot应用程序中，或者用Spring Boot已提供的服务器替换它。
- 消息传输：Spring Boot提供了不同的客户端，例如[RestClient](https://www.baeldung.com/spring-boot-restclient)(阻塞)或[WebClient](https://www.baeldung.com/spring-5-webclient)(非阻塞)，但遗憾的是它们无法配置和用于gRPC，因为 gRPC对阻塞和非阻塞调用都使用[自定义传输技术](https://github.com/grpc/grpc-java/tree/master?tab=readme-ov-file#transport)。
- 配置：由于gRPC带来了自己的技术，因此我们需要配置属性来以Spring Boot方式对其进行配置。

## 3. 示例项目

幸运的是，我们可以使用第三方Spring Boot Starters来帮助我们应对挑战，例如来自[LogNet](https://github.com/LogNet/grpc-spring-boot-starter)或[grpc生态系统](https://github.com/grpc-ecosystem/grpc-spring)项目的Starters。这两个Starters都易于集成，但后者同时支持提供者和消费者以及许多其他[集成功能](https://github.com/grpc-ecosystem/grpc-spring?tab=readme-ov-file#features)，因此我们选择它作为示例。

在此示例中，**我们仅使用一个Proto文件设计一个简单的HelloWorld API**：

```protobuf
syntax = "proto3";

option java_package = "cn.tuyucheng.taketoday.helloworld.stubs";
option java_multiple_files = true;

message HelloWorldRequest {
    // a name to greet, default is "World"
    optional string name = 1;
}

message HelloWorldResponse {
    string greeting = 1;
}

service HelloWorldService {
    rpc SayHello(stream HelloWorldRequest) returns (stream HelloWorldResponse);
}
```

如我们所见，我们使用了[双向流](https://www.baeldung.com/java-grpc-streaming)功能。

### 3.1 gRPC存根

由于提供者和消费者的存根是相同的，因此我们在[单独的、独立于Spring的项目](https://www.baeldung.com/grpc-introduction)中生成它们。这样做的好处是，项目的生命周期(包括protoc编译器配置和Java EE注解依赖项)可以与Spring Boot项目的生命周期隔离。

### 3.2 服务提供者

实现服务提供者非常简单。首先，我们需要添加Starter和存根项目的依赖项：

```xml
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-server-spring-boot-starter</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>cn.tuyucheng.taketoday.spring-boot-modules</groupId>
    <artifactId>helloworld-grpc-java</artifactId>
    <version>1.0.0</version>
</dependency>
```

无需包含Spring MVC或WebFlux，因为Starter依赖项带来了Shaded Netty服务器。我们可以在application.yml中对其进行配置，例如，通过配置服务器端口：

```yaml
grpc:
  server:
    port: 9090
```

然后，我们需要实现该服务并用@GrpcService标注它：

```java
@GrpcService
public class HelloWorldController extends HelloWorldServiceGrpc.HelloWorldServiceImplBase {

    @Override
    public StreamObserver<HelloWorldRequest> sayHello(
            StreamObserver<HelloWorldResponse> responseObserver
    ) {
        // ...
    }
}
```

### 3.3 服务消费者

对于服务消费者，我们需要添加Starter和存根依赖项：

```xml
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-client-spring-boot-starter</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>cn.tuyucheng.taketoday.spring-boot-modules</groupId>
    <artifactId>helloworld-grpc-java</artifactId>
    <version>1.0.0</version>
</dependency>
```

然后，我们在application.yml中配置与服务的连接：

```yaml
grpc:
  client:
    hello:
      address: localhost:9090
      negotiation-type: plaintext
```

“hello”这个名字是自定义的，这样，我们可以配置多个连接，并在将gRPC客户端注入到我们的Spring组件时引用这个名字：

```java
@GrpcClient("hello")
HelloWorldServiceGrpc.HelloWorldServiceStub stub;
```

## 4. 陷阱

使用Spring Boot实现和使用gRPC服务非常简单，但我们应该注意一些陷阱。

### 4.1 SSL握手

通过HTTP传输数据意味着发送未加密的信息，除非我们使用SSL。集成的Netty服务器默认不使用SSL，因此我们需要明确[配置](https://www.baeldung.com/netty-http2#sslcontext)它。

否则，对于本地测试，我们可以不保护连接。在这种情况下，我们需要配置消费者，如下所示：

```yaml
grpc:
  client:
    hello:
      negotiation-type: plaintext
```

消费者默认使用TLS，而提供者默认跳过SSL加密。**因此，消费者和提供者的默认设置并不匹配**。

### 4.2 不使用@Autowired进行消费者注入

我们通过向Spring组件注入客户端对象来实现消费者：

```java
@GrpcClient("hello")
HelloWorldServiceGrpc.HelloWorldServiceStub stub;
```

这是由[BeanPostProcessor](https://www.baeldung.com/spring-beanpostprocessor)实现的，是Spring内置依赖注入机制的补充。这意味着我们不能将@GrpcClient注解与@Autowired或构造函数注入结合使用；相反，我们只能使用字段注入。

我们只能通过使用配置类来分离注入：

```java
@Configuration
public class HelloWorldGrpcClientConfiguration {

    @GrpcClient("hello")
    HelloWorldServiceGrpc.HelloWorldServiceStub helloWorldClient;

    @Bean
    MyHelloWorldClient helloWorldClient() {
        return new MyHelloWorldClient(helloWorldClient);
    }
}
```

### 4.3 映射传输对象

当调用具有空值的Setter时，protoc生成的数据类型可能会失败：

```java
public HelloWorldResponse map(HelloWorldMessage message) {
    return HelloWorldResponse
        .newBuilder()
        .setGreeting( message.getGreeting() ) // might be null
        .build();
}
```

因此，我们需要在调用Setter之前进行空值检查。当我们使用映射框架时，我们需要配置映射器生成来执行此类空值检查。例如，[MapStruct](https://www.baeldung.com/mapstruct)映射器需要一些特殊配置：

```java
@Mapper(
    componentModel = "spring",
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS
)
public interface HelloWorldMapper {
    HelloWorldResponse map(HelloWorldMessage message);
}
```

### 4.4 测试

Starter不包含任何用于实现测试的特殊支持，即使是gRPC for Java项目也只对JUnit 4提供最低限度的支持，而[对JUnit 5则完全不支持](https://github.com/grpc/grpc-java/issues/5331)。

### 4.5 原生镜像

当我们想要构建[原生镜像](https://www.baeldung.com/spring-native-intro)时，目前还[没有对gRPC的支持](https://github.com/grpc-ecosystem/grpc-spring/issues/577)。由于客户端注入是通过反射完成的，因此如果不进行[额外配置](https://www.baeldung.com/spring-native-intro#extend-the-native-image-build-configuration)，这将无法实现。

## 5. 总结

在本文中，我们了解到我们可以在Spring Boot应用程序中轻松实现gRPC提供者和消费者。但是，我们应该注意，这会带来一些限制，例如缺少对测试和本机镜像的支持。