---
layout: post
title:  EnableEurekaClient与EnableDiscoveryClient
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Eureka
---

## 1. 简介

在本教程中，我们将研究[@EnableEurekaClient](https://www.baeldung.com/spring-cloud-netflix-eureka)与[@EnableDiscoveryClient](https://www.baeldung.com/spring-cloud-consul)的区别。

这两个都是在开发微服务时使用Spring Boot中的服务注册中心时使用的注解，这些注解成为其他客户端或微服务作为客户端发现的一部分在服务注册中心中注册的基础。

## 2. 微服务中的服务注册中心

在以分布式和松耦合为特征的微服务领域，我们面临着巨大的挑战，具体来说，随着服务数量的增加，维护准确的清单并确保它们之间的无缝通信变得越来越复杂。因此，手动跟踪服务健康和可用性被证明是一项资源密集型且容易出错的工作，这时，服务注册中心就显得尤为重要。

服务注册中心是一个保存可用服务信息的数据库，**它为服务注册、查找和管理提供了一个中心点**。

当微服务启动时，它会在服务注册中心中注册自己，其他需要通信的服务会将其用作查找或使用查找进行负载平衡，它还会通过删除和添加来动态更新服务实例记录。

## 3. @EnableDiscoveryClient注解

**@EnableDiscoveryClient是Spring Boot提供的更通用的注解，使其成为服务发现的灵活选择**。

我们不能将其视为一种契约，允许我们的应用程序现在和将来与不同的服务注册中心进行交互，它与[spring-cloud-commons](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies)依赖捆绑在一起，使其成为Spring Cloud服务发现机制的核心部分。

但是，它不能独立工作-我们需要为实际实现添加特定的依赖，无论是Eureka、Consul还是Zookeeper。

## 4. @EnableEurekaClient注解

Eureka最初由Netflix开发，提供强大的服务发现解决方案，Eureka Client是Eureka的基本组件。

具体来说，应用程序可以充当客户端，将自己注册到使用Eureka实现的服务注册中心中。此外，Spring Cloud在其上提供了一个有价值的抽象层，并与Spring Boot结合，大大简化了Eureka与微服务架构的集成，从而提高了开发人员的工作效率。

**因此，@EnableEurekaClient注解在这个过程中起着关键作用，更准确地说，它是用于服务作为客户端将自己注册到服务注册中心的注解之一**。

### 4.1 实现Eureka客户端

仅当我们在pom.xml文件中拥有[Eureka客户端依赖](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client)时，Eureka Client才有效：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

我们还需要在一文件中的dependencyManagement部分指定[spring-cloud-commons](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies)：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2021.0.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement> 
```

一旦注册，它会提供元数据，如主机、URL、健康URL等。服务注册中心还会从每个服务获取心跳信息，如果心跳未在可配置的时间表内发送，则该实例将从服务注册服务器中删除。

我们现在将检查配置Eureka客户端所需的基本Spring Boot属性，它们出现在application.properties或application.yml文件中：

```properties
spring.application.name=eurekaClient
server.port=8081
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
eureka.instance.prefer-ip-address=true
```

让我们理解上述属性：

- spring.application.name：这为我们的应用程序提供了一个名称，一个逻辑标识符(用于服务发现)。
- server.port：此客户端运行的端口号。
- eureka.client.service-url.defaultZone：这是客户端找到Eureka服务器的地方，本质上，它是服务注册中心的地址。因此，“defaultZone”是客户端知道如何注册和获取服务的地方。
- eureka.instance.prefer-ip-address：此属性规定Eureka客户端应注册其IP地址而不是其主机名。

### 4.2 设置Eureka服务器

在Eureka Client注册之前，我们还必须设置[Eureka Server](https://www.baeldung.com/spring-cloud-netflix-eureka#Eureka)。为此，我们需要一个带有@SpringBootApplication注解的Spring Boot应用程序，它是任何Spring Boot应用程序的基础，用于设置基本配置。

**因此，@EnableEurekaServer注解将我们的应用程序转换为Eureka服务器**。换句话说，它不仅仅是一个向注册中心注册的客户端；它本身就是注册中心。因此，此应用程序成为中央服务发现中心：

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

现在，让我们在application.properties文件中包含一些用于配置Eureka服务器的属性：

```properties
spring.application.name=eurekaServer
server.port=8761
eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

让我们仔细看看上述属性：

- spring.application.name：它为我们的应用程序提供了一个名称标签。
- server.port：指定Spring Boot应用程序监听传入请求的网络端口。
- eureka.instance.hostname：定义Eureka服务器实例在注册过程中公布的主机名，在此配置中，localhost表示服务器在本地计算机上执行。在生产环境中，此属性将填充服务器的完全限定域名或IP地址。
- eureka.client.register-with-eureka：设置为false时，它会指示Eureka服务器实例不要将自身注册到另一个Eureka服务器
- eureka.client.fetch-registry：设置为false时，它会阻止Eureka服务器实例从另一个Eureka服务器检索服务注册中心

一旦Eureka服务器运行起来，我们就可以运行Eureka客户端了。

## 5. 在@EnableEurekaClient和@EnableDiscoveryClient之间进行选择

@EnableDiscoveryClient是Spring Cloud Netflix在未来构建服务注册工具时放置在@EnableEurekaClient之上的抽象接口，另一方面，@EnableEurekaClient用于使用Eureka进行服务发现。

类似地，在Eureka、Consul、Zookeeper等其他实现中，我们可以有各种组合：

- @EnableDiscoveryClient与@EnableEurekaClient以及服务注册服务器作为@EnableEurekaServer。
- @EnableDiscoveryClient使用Consul和服务注册服务器作为Consul Server。

如果我们的服务注册中心是Eureka，那么@EnableEurekaClient和@EnableDiscoveryClient在技术上都可以为我们的客户端工作。但是，由于其简单性，通常建议使用@EnableEurekaClient。

**@EnableDiscoveryClient在我们希望客户端获得更多灵活性的情况下非常有用，因为它可以给予我们更多的控制权**。

还值得注意的是，我们经常会在配置类上看到@EnableDiscoveryClient注解，用于启用发现客户端。但这里有一个巧妙的技巧：如果我们使用Spring Cloud Starter，则不需要该注解。在这种情况下，默认情况下会启用发现客户端。此外，如果Spring Cloud在我们的类路径上找到Netflix Eureka客户端，它会自动为我们配置。

我们来总结一下它们的主要区别：

|                @EnableEurekaClient                |    @EnableDiscoveryClient   |
|:-------------------------------------------------:|:---------------------------:|
|                仅使用Eureka启用服务注册和发现                 |      更通用，可与多个服务注册中心配合使用     |
|                 仅支持Netflix Eureka                 |  支持Eureka，Consul，Zookeeper等 |
|                      一个具体的实现                      |           一个接口或者契约          |
|需要spring-cloud-starter-netflix-eureka-client依赖才能工作 | Spring Cloud依赖的一部分。无需任何特定依赖 |



## 6. 总结

在本文中，我们了解了@EnableEurekaClient和@EnableDiscoveryClient如何与Eureka配合使用。但是，如果我们确定要使用Eureka，那么@EnableEurekaClient通常是更简单、更直接的选择。

另一方面，@EnableDiscoveryClient更加通用，如果我们认为将来可能会使用不同的服务注册中心，或者我们只是想要更多的灵活性和控制，那么它是一个不错的选择。