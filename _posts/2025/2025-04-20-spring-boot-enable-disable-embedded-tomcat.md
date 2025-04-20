---
layout: post
title:  在Spring Boot中使用Profile启用或禁用嵌入式Tomcat
category: springboot
copyright: springboot
excerpt: Tomcat
---

## 1. 概述

当我们想要在Spring Boot应用程序中启用或禁用嵌入式[Tomcat服务器](https://www.baeldung.com/tomcat)时，我们需要根据应用程序的需求考虑不同的方法。默认情况下，Spring Boot提供了一个嵌入式Tomcat服务器，但在某些情况下，我们可能想要禁用它。

**对于需要嵌入式服务器的应用程序，我们可以使用默认配置。但是，对于不公开Web端点或需要作为后台服务运行的应用程序，禁用Tomcat可以优化资源使用率**。

在本教程中，我们将探讨何时启用或禁用嵌入式Tomcat服务器以及如何配置Spring Boot Profile以动态实现这一点。

## 2. 了解Spring Boot中的嵌入式Tomcat

Spring Boot通过在应用程序的可执行[JAR文件](https://www.baeldung.com/java-create-jar)中捆绑嵌入式Tomcat服务器来简化应用程序的部署，这种方法无需安装和配置外部Tomcat实例，从而提高了开发和部署的效率。

Spring Boot使用[Spring Boot Starters](https://www.baeldung.com/spring-boot-starters)来包含嵌入式Tomcat所需的依赖，**默认启动器spring-boot-starter-web会在Tomcat出现在Classpath中时自动配置并初始化Tomcat**。

### 2.1 嵌入式Tomcat的优势

Spring Boot的嵌入式Tomcat服务器具有以下几个优点：

- **简化部署**：无需安装外部Tomcat服务器
- **自包含应用程序**：应用程序可以打包为JAR文件并在任何地方运行
- **自动配置**：Spring Boot根据依赖自动配置Tomcat
- **灵活性**：可以轻松替换为[Jetty](https://www.baeldung.com/jetty-embedded)或Undertow等其他嵌入式服务器

### 2.2 何时禁用Tomcat服务器

虽然嵌入式Tomcat很有用，但在某些情况下禁用它会对我们有利：

- **非Web应用程序**：不处理HTTP请求的应用程序，例如CLI工具或批处理作业
- **具有替代服务器的微服务**：某些服务可能使用专用的Web服务器，例如[Nginx](https://www.baeldung.com/nginx-forward-proxy)
- **资源优化**：禁用Tomcat可减少内存和CPU使用率

## 3. 配置Spring Boot Profile

Spring Boot提供了spring.profiles.active属性来定义特定于环境的配置，我们可以创建不同的基于Profile的配置来控制是否启用嵌入式Tomcat服务器。

为了定义Profile，我们通常创建单独的Profile，例如：

- application-dev.properties(用于启用Tomcat的开发)
- application-batch.properties(用于不启用Tomcat的批处理)

## 4. 使用Profile禁用嵌入式Tomcat

Spring Boot根据spring.main.web-application-type属性确定是否启用嵌入式Web服务器，我们可以将其设置为NONE来禁用嵌入式Tomcat。

为了在特定于Profile的配置中执行此操作，我们修改application-batch.properties文件：

```properties
spring.main.web-application-type=NONE
```

当此Profile处于激活状态时，Spring Boot将不会启动Tomcat，而是将应用程序视为非Web服务。

或者，我们可以使用[YAML](https://www.baeldung.com/spring-yaml)配置此设置：

```yaml
spring:
  main:
    web-application-type: NONE
```

## 5. 不同Profile的示例配置

让我们用两个Profile配置一个应用程序：

- 开发Profile(dev)：Tomcat启用(默认设置)
- 批处理Profile(batch)：Tomcat禁用

为了确保我们的嵌入式Tomcat服务器正常启动，让我们在application-dev.properties文件中设置属性：

```properties
spring.main.web-application-type=SERVLET
```

为了在批处理禁用嵌入式Tomcat服务器，我们需要在application-batch.properties文件中设置属性：

```properties
spring.main.web-application-type=NONE
```

## 6. 在Profile之间切换

一旦我们定义了多个Profile，我们就可以通过application.properties文件激活所需的Profile：

```properties
spring.profiles.active=batch
```

或者，我们可以将其作为命令行参数传递：

```shell
java -Dspring.profiles.active=batch -jar myapp.jar
```

这种灵活性使我们能够根据需要在开发、测试或生产部署期间在支持Web模式和非Web模式之间切换。

## 7. 总结

Spring Boot允许使用Profile灵活配置嵌入式Tomcat服务器，通过利用spring.main.web-application-type，我们可以在非Web应用程序需要时禁用Tomcat，从而优化资源使用和部署配置，使用基于Profile的设置或动态Java逻辑可确保我们的应用程序无缝适应不同的环境。