---
layout: post
title:  使用OAuth2和JWT在Zuul中处理安全性
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 简介

简而言之，微服务架构允许我们将系统和API分解为一组自包含的服务，这些服务可以完全独立部署。

虽然从持续部署和管理的角度来看这很棒，但在API可用性方面，它很快就会变得复杂。由于需要管理不同的端点，依赖应用程序将需要管理CORS(跨源资源共享)和一组不同的端点。

Zuul是一项边缘服务，它允许我们将传入的HTTP请求路由到多个后端微服务。首先，这对于为后端资源的消费者提供统一的API非常重要。

基本上，Zuul允许我们通过位于所有服务前面并充当代理来统一所有服务，它接收所有请求并将其路由到正确的服务。对于外部应用程序，我们的API显示为统一的API表面积。

在本教程中，我们将讨论如何将其与[OAuth 2.0和JWT](https://www.baeldung.com/spring-security-oauth-jwt)结合使用，作为保护我们Web服务的第一线。具体来说，我们将使用[密码授权](https://oauth.net/2/grant-types/password/)流程来获取受保护资源的访问令牌。

一个简短但重要的注意事项是，我们仅使用密码授予流程来探索简单的场景；大多数客户端更有可能在生产场景中使用授权授予流程。

## 2. 添加Zuul Maven依赖

首先将Zuul添加到[我们的项目](https://github.com/Baeldung/spring-security-oauth)中，我们通过添加[spring-cloud-starter-netflix-zuul](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-zuul)工件来实现这一点：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    <version>2.0.2.RELEASE</version>
</dependency>
```

## 3. 启用Zuul

我们希望通过Zuul路由的应用程序包含一个授予访问令牌的OAuth 2.0授权服务器和一个接收令牌的资源服务器，这些服务位于两个独立的端点上。

我们希望为这些服务的所有外部客户端提供一个端点，并将不同的路径分支到不同的物理端点。为此，我们将引入Zuul作为边缘服务。

为此，我们将创建一个名为GatewayApplication的新Spring Boot应用程序。然后，我们将使用@EnableZuulProxy注解简单地装饰此应用程序类，这会生成一个Zuul实例：

```java
@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

## 4. 配置Zuul路由

在继续之前，我们需要配置一些Zuul属性。首先要配置的是Zuul监听传入连接的端口，这需要更改/src/main/resources/application.yml文件：

```yaml
server:
    port: 8080
```

现在到了有趣的部分，配置Zuul将转发到的实际路由。为此，我们需要注意以下服务、它们的路径以及它们监听的端口。

**授权服务器部署在**：[http://localhost:8081/spring-security-oauth-server/oauth](http://localhost:8081/spring-security-oauth-server/oauth)

**资源服务器部署在**：[http://localhost:8082/spring-security-oauth-resource](http://localhost:8082/spring-security-oauth-resource)

授权服务器是OAuth身份提供者，它用于向资源服务器提供授权令牌，资源服务器则提供一些受保护的端点。

授权服务器向客户端提供访问令牌，然后客户端使用该令牌代表资源所有者向资源服务器执行请求，快速浏览[OAuth术语](https://oauth2.thephpleague.com/terminology/)将有助于我们记住这些概念。

现在让我们将一些路由映射到每个服务：

```yaml
zuul:
    routes:
        spring-security-oauth-resource:
            path: /spring-security-oauth-resource/**
            url: http://localhost:8082/spring-security-oauth-resource
        oauth:
            path: /oauth/**
            url: http://localhost:8081/spring-security-oauth-server/oauth

```

此时，任何到达localhost:8080/oauth/\*\*上的Zuul的请求都将被路由到在端口8081上运行的授权服务，任何到达localhost:8080/spring-security-oauth-resource/\*\*的请求都将被路由到在8082上运行的资源服务器。

## 5. 保护Zuul外部流量路径

尽管我们的Zuul边缘服务现在可以正确路由请求，但它没有进行任何授权检查。位于/oauth/\*后面的授权服务器为每次成功的身份验证创建一个JWT。当然，它可以匿名访问。

另一方面，位于/spring-security-oauth-resource/\*\*的资源服务器应始终使用JWT进行访问，以确保授权客户端正在访问受保护的资源。

首先，我们将配置Zuul，以便将JWT传递给其背后的服务。在本例中，这些服务本身需要验证令牌。

我们通过添加sensitiveHeaders:Cookie,Set-Cookie来实现这一点。

这样就完成了我们的Zuul配置：

```yaml
server:
    port: 8080
zuul:
    sensitiveHeaders: Cookie,Set-Cookie
    routes:
        spring-security-oauth-resource:
            path: /spring-security-oauth-resource/**
            url: http://localhost:8082/spring-security-oauth-resource
        oauth:
            path: /oauth/**
            url: http://localhost:8081/spring-security-oauth-server/oauth
```

解决此问题后，我们需要处理边缘授权问题。目前，Zuul在将JWT传递给下游服务之前不会对其进行验证，这些服务将自行验证JWT。但理想情况下，我们希望边缘服务先进行验证，并在任何未经授权的请求深入到我们的架构之前将其拒绝。

**让我们设置Spring Security以确保在Zuul中检查授权**。

首先，我们需要将Spring Security依赖引入项目，我们需要[spring-security-oauth2](https://mvnrepository.com/artifact/org.springframework.security.oauth/spring-security-oauth2)和[spring-security-jwt](https://mvnrepository.com/artifact/org.springframework.security/spring-security-jwt)：

```xml
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
    <version>1.0.9.RELEASE</version>
</dependency>
```

现在让我们通过扩展ResourceServerConfigurerAdapter来为想要保护的路由编写一个配置：

```java
@Configuration
@Configuration
@EnableResourceServer
public class GatewayConfiguration extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(final HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/oauth/**")
                .permitAll()
                .antMatchers("/**")
                .authenticated();
    }
}
```

GatewayConfiguration类定义Spring Security应如何处理通过Zuul传入的HTTP请求。在configure方法中，我们首先使用antMatchers匹配最严格的路径，然后通过permitAll允许匿名访问。

也就是说，进入/oauth/\*\*的所有请求都应被允许通过，而无需检查任何授权令牌。这是有道理的，因为这是生成授权令牌的路径。

接下来，我们将所有其他路径与/\*\*进行匹配，并通过对authenticated的调用坚持要求所有其他调用都应包含访问令牌。

## 6. 配置用于JWT验证的密钥

现在配置已到位，所有路由到/oauth/\*\*路径的请求都将被匿名允许通过，而所有其他请求都需要身份验证。

不过，我们这里还缺少一件事，那就是验证JWT是否有效所需的实际密钥。为此，我们需要提供用于签署JWT的密钥(在本例中是对称的)。我们可以使用[spring-security-oauth2-autoconfigure](https://mvnrepository.com/artifact/org.springframework.security.oauth.boot/spring-security-oauth2-autoconfigure)，而不是手动编写配置代码。

让我们首先将依赖添加到项目中：

```xml
<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
```

接下来，我们需要在application.yaml文件中添加几行配置来定义用于签署JWT的密钥：

```yaml
security:
    oauth2:
        resource:
            jwt:
                key-value: 123
```

key-value:123这一行设置授权服务器用于签署JWT的对称密钥，spring-security-oauth2-autoconfigure将使用此密钥来配置令牌解析。

需要注意的是，在生产系统中，**我们不应该使用在应用程序源代码中指定的对称密钥**，这自然需要在外部进行配置。

## 7. 测试边缘服务

### 7.1 获取访问令牌

现在让我们使用一些curl命令来测试我们的Zuul边缘服务的行为。

首先，我们将了解如何使用[密码授权](https://oauth.net/2/grant-types/password/)从授权服务器获取新的JWT。

这里我们**使用用户名和密码来交换访问令牌**，在本例中，我们使用'john'作为用户名，'123'作为密码：

```shell
curl -X POST \
  http://localhost:8080/oauth/token \
  -H 'Authorization: Basic Zm9vQ2xpZW50SWRQYXNzd29yZDpzZWNyZXQ=' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&password=123&username=john'
```

此调用会产生一个JWT令牌，然后我们可以使用它来针对资源服务器发出经过身份验证的请求。

注意“Authorization: Basic...”标头字段，它用于告知授权服务器哪个客户端正在连接它。

对于客户端(在本例中为cURL请求)，用户名和密码对于用户来说就是：

```json
{
    "access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpX...",
    "token_type":"bearer",
    "refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpX...",
    "expires_in":3599,
    "scope":"foo read write",
    "organization":"johnwKfc",
    "jti":"8e2c56d3-3e2e-4140-b120-832783b7374b"
}
```

### 7.2 测试资源服务器请求

然后，我们可以使用从授权服务器检索到的JWT对资源服务器执行查询：

```shell
curl -X GET \
curl -X GET \
  http:/localhost:8080/spring-security-oauth-resource/users/extra \
  -H 'Accept: application/json, text/plain, */*' \
  -H 'Accept-Encoding: gzip, deflate' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXV...' \
  -H 'Cache-Control: no-cache' \
```

Zuul边缘服务现在将在路由到资源服务器之前验证JWT。

然后从JWT中提取关键字段，并在响应请求之前检查更细粒度的授权：

```json
{
    "user_name":"john",
    "scope":["foo","read","write"],
    "organization":"johnwKfc",
    "exp":1544584758,
    "authorities":["ROLE_USER"],
    "jti":"8e2c56d3-3e2e-4140-b120-832783b7374b",
    "client_id":"fooClientIdPassword"
}
```

## 8. 跨层安全

需要注意的是，JWT在传递到资源服务器之前会由Zuul边缘服务进行验证。如果JWT无效，则请求将在边缘服务边界被拒绝。

另一方面，如果JWT确实有效，则请求将传递到下游。然后，资源服务器再次验证JWT，并提取关键字段，例如用户范围、组织(在本例中为自定义字段)和权限，它使用这些字段来决定用户可以做什么和不能做什么。

需要明确的是，在很多架构中，我们实际上并不需要验证JWT两次-这是你必须根据流量模式做出的决定。

例如，在某些生产项目中，可以直接访问单个资源服务器，也可以通过代理访问-我们可能希望在两个地方验证令牌。在其他项目中，流量可能仅通过代理，在这种情况下验证令牌就足够了。

## 9. 总结

正如我们所见，Zuul提供了一种简单、可配置的方式来抽象和定义服务的路由。与Spring Security结合使用，它允许我们在服务边界授权请求。