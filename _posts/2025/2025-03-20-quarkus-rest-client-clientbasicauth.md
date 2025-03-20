---
layout: post
title:  在Quarkus REST客户端中使用@ClientBasicAuth
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

Quarkus是一个基于Java的框架，用于构建基于[Jakarta EE](https://www.baeldung.com/jakarta-ee-10)和[MicroProfile](https://www.baeldung.com/eclipse-microprofile)的应用程序，主要围绕[REST](https://www.baeldung.com/rest-vs-graphql-vs-grpc)服务。为了更轻松地访问这些服务，Quarkus提供了一个[REST客户端](https://download.eclipse.org/microprofile/microprofile-rest-client-3.0/apidocs/org/eclipse/microprofile/rest/client/RestClientBuilder.html)，允许我们使用类型安全的代理对象访问此类REST服务。

**当REST资源受到保护时，我们需要进行身份验证**。在REST/HTTP中，这通常通过发送包含我们凭据的HTTP标头来完成。不幸的是，REST客户端API不包含任何提供安全详细信息的方法，甚至不包含带有请求的HTTP标头。

在本教程中，我们将介绍如何通过[Quarkus](https://www.baeldung.com/quarkus-io)(MicroProfile) REST客户端访问受保护的REST服务。**Quarkus提供了一种提供基本身份验证凭据的简单方法：@ClientBasicAuth注解**。

## 2. 设置受保护的服务

**让我们首先使用Quarkus来设置受保护的服务**。

考虑以下REST服务接口：

```java
@Path("/hello")
@RequestScoped
public interface MyService {

    @GET
    @Produces(TEXT_PLAIN)
    String hello();
}
```

然后我们来实现MyService：

```java
public class MyServiceImpl implements MyService {

    @Override
    @RolesAllowed("admin")
    public String hello() {
        return "Hello from Quarkus REST";
    }
}
```

[@RolesAllowed](https://www.baeldung.com/java-rbac-quarkus)注解位于实现类上，而不是接口上。令人惊讶的是，这是必需的，因为@RolesAllowed注解不是从接口继承的。然而，这可能不是那么糟糕，因为角色名称可以被视为不应公开的内部细节。

默认情况下，当我们启用安全性时，Quarkus会激活基本身份验证机制。但是，我们必须提供一个[身份存储](https://www.baeldung.com/java-ee-8-security#identity-store)(Quarkus中的IdentityProvider)。基本上，我们可以通过在[类路径](https://www.baeldung.com/java-classpath-sourcepath#classpath)上提供它来实现这一点。在这种情况下，我们将使用一个简单的基于属性文件的身份存储，方法是将以下内容放入pom.xml中：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-elytron-security-properties-file</artifactId>
</dependency>
```

这将激活安全性(通过其[传递依赖](https://www.baeldung.com/maven-dependency-scopes)io.quarkus:quarkus-elytron-security)，并将上述身份存储放在类路径上。我们可以使用application.properties中的属性将用户及其密码和角色添加到此存储中：

```properties
quarkus.http.auth.basic=true
quarkus.security.users.embedded.enabled=true
quarkus.security.users.embedded.plain-text=true

quarkus.security.users.embedded.users.john=secret1
quarkus.security.users.embedded.roles.john=admin,user
```

## 3. RestClientBuilder和@ClientBasicAuth

**让我们看看如何使用RestClientBuilder调用受保护的服务**。

这是一个类型安全的调用，不使用凭证：

```java
MyService myService = RestClientBuilder.newBuilder()
    .baseUri(URI.create("http://localhost:8081"))
    .build(MyService.class);

 myService.hello();
```

这将为REST服务创建一个代理，然后我们可以像调用本地类一样调用它。

提供安全凭证的一种方法是使用@ClientHeaderParam标注原始Jakarta REST接口，另一种方法是从此原始接口扩展并将注解放在那里。在这两种情况下，我们都必须使用注解指定的回调方法手动(通过代码)生成正确的标头，由于我们使用基本身份验证，因此我们可以利用@ClientBasicAuth注解。

**为了使用@ClientBasicAuth提供用于基本身份验证的用户名/密码凭据，我们创建了特定于给定用户的新接口类型**。因此，用户名和密码没有动态性。以这种方式创建的每种类型都对应于特定用户，当应用程序使用一个或多个静态定义的系统用户访问远程服务时，这很有用。

让我们创建接口类型：

```java
@ClientBasicAuth(username = "john", password = "secret1")
public interface MyServiceJohnRightCredentials extends MyService {
}
```

随后，我们传入带有凭证的类型而不是原始类型：

```java
MyService myService = RestClientBuilder.newBuilder()
    .baseUri(URI.create("http://localhost:8081"))
    .build(MyServiceJohnRightCredentials.class);

myService.hello();
```

**使用上面的代码，Quarkus RestClientBuilder会生成正确的标头，以使用基本身份验证访问REST服务**。值得注意的是，我们在这里使用常量以简化操作。实际上，我们可以使用${john.password}之类的表达式来引用配置属性。

## 4. 总结

在本文中，我们了解了如何设置受基本身份验证保护的REST服务，以及如何随后使用@ClientBasicAuthentication注解访问此类服务。

我们看到，使用此@ClientBasicAuthentication注解我们不必修改现有的REST接口。但是，我们还发现我们以这种方式设置了静态(预定义)用户；@ClientBasicAuthentication不适合动态输入。