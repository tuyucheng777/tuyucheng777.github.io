---
layout: post
title:  为Spring Cloud Gateway配置CORS策略
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Gateway
---

## 1. 概述

[跨域资源共享](https://www.baeldung.com/cs/cors-preflight-requests)(CORS)是一种基于浏览器的应用程序安全机制，允许一个域的网页访问另一个域的资源，浏览器实施同源访问策略来限制任何跨域应用程序访问。

此外，Spring还提供了一流的[支持](https://docs.spring.io/spring-framework/reference/web/webmvc-cors.html)，可以在任何Spring、Spring Boot Web和Spring Cloud Gateway应用程序中轻松配置CORS。

在本文中，我们将学习如何使用后端API设置[Spring Cloud Gateway](https://www.baeldung.com/spring-cloud-gateway)应用程序。此外，我们将访问网关API并调试常见的CORS相关错误。

然后，我们将使用[Spring CORS](https://www.baeldung.com/spring-cors)支持配置Spring网关API。

## 2. 使用Spring Cloud Gateway实现API网关

假设我们需要构建一个Spring Cloud网关服务来公开后端REST API。

### 2.1 实现后端REST API

我们的后端应用程序将有一个返回用户数据的端点。

首先，让我们对User类进行建模：
```java
public class User {
    private long id;
    private String name;

    //standard getters and setters
}
```

接下来，我们将使用getUser端点实现UserController：
```java
@GetMapping(path = "/user/{id}")
public User getUser(@PathVariable("id") long userId) {
    LOGGER.info("Getting user details for user Id {}", userId);
    return userMap.get(userId);
}
```

### 2.2 实现Spring Cloud Gateway服务

现在让我们使用[Spring Cloud Gateway](https://www.baeldung.com/spring-cloud-gateway-integrate-openapi)支持实现API网关服务。

首先，我们将包含[spring-cloud-starter-gateway](https://mvnrepository.com/search?q=spring-cloud-starter-gateway)依赖：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>4.1.5</version
</dependency>
```

### 2.3 配置API路由

我们可以使用Spring Cloud Gateway[路由选项](https://www.baeldung.com/spring-cloud-gateway#dynamic-routing)公开用户服务端点。

我们将使用/user路径配置谓词，并使用后端URI http://<hostname\>:<port\>设置uri属性：
```yaml
spring:
    cloud:
        gateway:
            routes:
                -  id: user_service_route
                   predicates:
                       - Path=/user/**
                   uri: http://localhost:8081
```

## 3. 测试Spring Gateway API

现在我们将使用cURL命令和浏览器窗口从终端测试Spring网关服务。

### 3.1 使用cURL测试网关API

让我们运行两个服务，用户和网关：
```shell
$ java -jar ./spring-backend-service/target/spring-backend-service-1.0.0.jar
```

```shell
$ java -jar ./spring-cloud-gateway-service/target/spring-cloud-gateway-service-1.0.0.jar
```

现在，让我们使用网关服务URL访问/user端点：

```shell
$ curl -v 'http://localhost:8080/user/100001'
```

```text
< HTTP/1.1 200 OK
< Content-Type: application/json
{"id":100001,"name":"User1"}
```

通过上述测试，我们能够获得后端API响应。

### 3.2 使用浏览器控制台进行测试

为了在浏览器环境中进行实验，我们将打开前端应用程序(例如https://www.tuyucheng.com)，并使用浏览器支持的开发人员工具选项。

我们将使用Javascript [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)函数从不同的来源URL调用API：
```javascript
fetch("http://localhost:8080/user/100001")
```

![](/assets/images/2025/springcloud/springcloudgateawayconfigurecorspolicy01.png)

从上面我们可以看出，由于CORS错误，API请求失败。

我们将从浏览器的Network选项卡进一步调试API请求：
```text
OPTIONS /user/100001 HTTP/1.1
Access-Control-Request-Method: GET
Access-Control-Request-Private-Network: true
Connection: keep-alive
Host: localhost:8080
Origin: https://www.tuyucheng.com
```

另外，让我们验证一下API响应：
```text
HTTP/1.1 403 Forbidden
...
content-length: 0
```

上述请求失败，因为网页URL的方案、域和端口与网关API不同，**浏览器期望服务器包含Access-Control-Allow-Origin标头**，但却收到错误。

默认情况下，由于来源不同，**Spring会在[预检](https://www.baeldung.com/cs/cors-preflight-requests#2-non-simple-requests)OPTIONS请求中返回Forbidden 403错误**。

接下来，我们将使用Spring Cloud Gateway支持的[CORS配置](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/cors-configuration.html)来修复该错误。

## 4. 在API网关中配置CORS策略

我们现在将配置CORS策略以允许不同的来源访问网关API。

让我们使用globalcors属性配置CORS访问策略：
```yaml
spring:
    cloud:
        gateway:
            globalcors:
                corsConfigurations:
                    '[/**]':
                        allowedOrigins: "https://www.tuyucheng.com"
                        allowedMethods:
                            - GET
                        allowedHeaders: "*"
```

我们应该注意，**globalcors属性会将CORS策略应用于所有路由端点**。

或者，我们可以为每个API路由配置CORS策略：
```yaml
spring:
    cloud:
        gateway:
            routes:
                -  id: user_service_route
                    ....
                   metadata:
                       cors:
                           allowedOrigins: 'https://www.tuyucheng.com,http://localhost:3000'
                           allowedMethods:
                               - GET
                               - POST
                           allowedHeaders: '*'
```

allowedOrigins字段可以配置为特定域名，或逗号分隔的域名，或设置为\*通配符以允许任何跨域访问。同样，可以将allowedMethods和allowedHeaders属性配置为特定值或使用\*通配符。

另外，我们可以使用替代的allowedOriginsPattern配置来为跨域模式匹配提供更多的灵活性：
```text
allowedOriginPatterns:
  - https://*.example1.com
  - https://www.example2.com:[8080,8081]
  - https://www.example3.com:[*]
```

与allowedOrigins属性不同，**allowedOriginsPattern允许在URL的任何部分(包括方案、域名和端口号)中使用\*通配符进行模式匹配**。此外，我们还可以在括号内指定逗号分隔的端口号，但是，allowedOriginsPattern属性不支持任何正则表达式。

现在，让我们在浏览器的控制台窗口中重新验证用户API：

![](/assets/images/2025/springcloud/springcloudgateawayconfigurecorspolicy02.png)

我们现在从API网关获得HTTP 200响应。

另外，让我们确认OPTIONS API响应中的Access-Control-Allow-Origin标头：
```text
HTTP/1.1 200 OK
...
Access-Control-Allow-Origin: https://www.tuyucheng.com
Access-Control-Allow-Methods: GET
content-length: 0
```

我们应该注意，建议配置一组有限的允许来源以提供最高级别的安全性。

默认情况下，CORS规范不允许跨域请求中包含任何cookie或CSRF令牌。但是，我们可以将allowedCredentials属性设置为true来启用它。此外，当在allowedOrigins和allowedHeaders属性中使用\*通配符时，allowedCredentials将不起作用。 

## 5. 总结

在本文中，我们学习了如何使用Spring Cloud Gateway支持实现网关服务，在从浏览器控制台测试API时，我们还遇到了一个常见的CORS错误。

最后，我们演示了如何通过使用allowedOrigins和allowedMethods属性配置应用程序来修复CORS错误。