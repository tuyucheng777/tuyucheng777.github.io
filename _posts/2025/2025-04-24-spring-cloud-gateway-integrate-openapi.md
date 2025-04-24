---
layout: post
title:  将OpenAPI与Spring Cloud Gateway集成
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Gateway
---

## 1. 概述

文档是构建任何健壮REST API的重要组成部分，我们可以基于OpenAPI规范实现API文档，并在Spring应用程序中的Swagger UI中将其可视化。

此外，由于可以使用API网关公开API端点，我们还需要将后端服务的[OpenAPI](https://www.baeldung.com/spring-rest-openapi-documentation)文档与网关服务集成，网关服务将提供所有API文档的统一视图。

在本文中，我们将学习如何在Spring应用程序中集成OpenAPI。此外，我们将使用[Spring Cloud Gateway](https://www.baeldung.com/spring-cloud-gateway)服务公开后端服务的API文档。

## 2. 示例应用程序

假设我们需要构建一个简单的微服务来获取一些数据。

### 2.1 Maven依赖

首先，我们将包含[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.3.2</version>
</dependency>
```

### 2.2 实现REST API

我们的后端应用程序将有一个返回产品数据的端点。

首先，让我们对Product类进行建模：
```java
public class Product {
    private long id;
    private String name;
    //standard getters and setters
}
```

接下来，我们将使用getProduct端点实现ProductController：
```java
@GetMapping(path = "/product/{id}")
public Product getProduct(@PathVariable("id") long productId){
    LOGGER.info("Getting Product Details for Product Id {}", productId);
    return productMap.get(productId);
}
```

## 3. 将Spring应用程序与OpenAPI集成

OpenAPI 3.0规范可以使用springdoc-openapi Starter项目与[Spring Boot 3](https://www.baeldung.com/spring-boot-3-spring-6-new)集成。

### 3.1 Springdoc依赖

Spring Boot 3.x要求我们使用[springdoc-openapi-starter-webmvc-ui](https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-starter-webmvc-ui)依赖的[版本2](https://github.com/springdoc/springdoc-openapi/releases/tag/v2.6.0)：
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

### 3.2 配置OpenAPI定义

我们可以使用一些Swagger注解自定义OpenAPI定义细节，如标题、描述和版本。

我们将使用一些属性配置@OpenAPI Bean，并设置@OpenAPIDefinition注解：
```java
@OpenAPIDefinition
@Configuration
public class OpenAPIConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .servers(List.of(new Server().url("http://localhost:8080")))
                .info(new Info().title("Product Service API").version("1.0.0"));
    }
}
```

### 3.3 配置OpenAPI和Swagger UI路径

可以使用springdoc-openapi配置自定义OpenAPI和Swagger UI默认路径。

我们将在springdoc-openapi属性中包含产品特定路径：
```yaml
springdoc:
    api-docs:
        enabled: true
        path: /product/v3/api-docs
    swagger-ui:
        enabled: true
        path: /product/swagger-ui.html
```

**我们可以在任何环境中通过将enabled标志设置为false来禁用OpenAPI的api-docs和Swagger UI功能**：
```yaml
springdoc:
    api-docs:
        enabled: false
    swagger-ui:
        enabled: false
```

### 3.4 添加API摘要

我们可以记录API摘要和有效负载详细信息，并包含任何与安全相关的信息。

让我们在ProductController类中包含API操作摘要详细信息：
```java
@Operation(summary = "Get a product by its id")
@ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Found the product",
                content = { @Content(mediaType = "application/json",
                        schema = @Schema(implementation = Product.class)) }),
        @ApiResponse(responseCode = "400", description = "Invalid id supplied",
                content = @Content),
        @ApiResponse(responseCode = "404", description = "Product not found",
                content = @Content) })
@GetMapping(path = "/product/{id}")
public Product getProduct(@Parameter(description = "id of product to be searched")
                          @PathVariable("id") long productId){}
```

在上面的代码中，我们设置了API操作摘要以及API请求和响应参数描述。

由于后端服务已经集成了OpenAPI，现在我们来实现一个API网关服务。

## 4. 使用Spring Cloud Gateway实现API网关

现在让我们使用Spring Cloud Gateway支持实现API网关服务，API网关服务将向我们的用户公开产品API。

### 4.1 Spring Cloud Gateway依赖

首先，我们将包含[spring-cloud-starter-gateway](https://mvnrepository.com/search?q=spring-cloud-starter-gateway)依赖：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>4.1.5</version>
</dependency>
```

### 4.2 配置API路由

我们可以使用Spring Cloud Gateway路由选项公开产品服务端点。

我们将使用/product路径配置谓词，并使用后端URI http://<hostname\>:<port\>设置uri属性：
```yaml
spring:
    cloud:
        gateway:
            routes:
                -   id: product_service_route
                    predicates:
                        - Path=/product/**
                    uri: http://localhost:8081
```

我们应该注意，**在任何投入生产的应用程序中，Spring Cloud Gateway都应该路由到后端服务的负载均衡器URL**。

### 4.3 测试Spring Gateway API

让我们运行产品和网关这两项服务：
```shell
$ java -jar ./spring-backend-service/target/spring-backend-service-1.0.0.jar
```

```shell
$ java -jar ./spring-cloud-gateway-service/target/spring-cloud-gateway-service-1.0.0.jar
```

现在，让我们使用网关服务URL访问/product端点：
```shell
$ curl -v 'http://localhost:8080/product/100001'
```

```text
< HTTP/1.1 200 OK
< Content-Type: application/json
{"id":100001,"name":"Apple"}
```

通过上述测试，我们能够获得后端API响应。

## 5. 将Spring Gateway服务与OpenAPI集成

现在我们可以将Spring Gateway应用程序与OpenAPI文档集成，就像在产品服务中所做的那样。

### 5.1 springdoc-openapi依赖

我们将包含[springdoc-openapi-starter-webflux-ui](https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-starter-webflux-ui/2.6.0)依赖，而不是springdoc-openapi-starter-webmvc-ui依赖：
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webflux-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

我们应该注意，**Spring Cloud Gateway需要webflux-ui依赖，因为它基于Spring[WebFlux](https://www.baeldung.com/spring-webflux)项目**。

### 5.2 配置OpenAPI定义

让我们使用一些与摘要相关的详细信息来配置一个OpenAPI Bean：
```java
@OpenAPIDefinition
@Configuration
public class OpenAPIConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI().info(new Info()
                .title("API Gateway Service")
                .description("API Gateway Service")
                .version("1.0.0"));
    }
}
```

### 5.3 配置OpenAPI和Swagger UI路径

我们将在Gateway服务中自定义OpenAPI api-docs.path和swagger-ui.urls属性：
```yaml
springdoc:
    api-docs:
        enabled: true
        path: /v3/api-docs
    swagger-ui:
        enabled: true
        config-url: /v3/api-docs/swagger-config
        urls:
            -   name: gateway-service
                url: /v3/api-docs
```

### 5.4 包含OpenAPI URL引用

要从网关服务访问产品服务api-docs端点，我们需要在上面的配置中添加它的路径。

我们将在上面的springdoc.swagger-ui.urls属性中包含/product/v3/api-docs路径：
```yaml
springdoc:
    swagger-ui:
        urls:
            -   name: gateway-service
                url: /v3/api-docs
            -   name: product-service
                url: /product/v3/api-docs
```

## 6. 在API网关应用程序中测试Swagger UI

当我们运行两个应用程序时，我们可以通过导航到[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)来查看Swagger UI中的API文档：

![](/assets/images/2025/springcloud/springcloudgatewayintegrateopenapi01.png)

现在，我们将从右上角下拉菜单访问产品服务api-docs：

![](/assets/images/2025/springcloud/springcloudgatewayintegrateopenapi02.png)

从上面的页面，我们可以查看和访问产品服务API端点。

我们可以通过访问[http://localhost:8080/product/v3/api-docs](http://localhost:8080/product/v3/api-docs)来访问JSON格式的产品服务API文档。

## 7. 总结

在本文中，我们学习了如何使用springdoc-openapi支持在Spring应用程序中实现OpenAPI文档。

我们还了解了如何在Spring Cloud Gateway服务中公开后端API。

最后，我们演示了如何使用Spring Gateway服务Swagger UI页面访问OpenAPI文档。