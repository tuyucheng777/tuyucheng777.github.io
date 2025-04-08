---
layout: post
title:  Spring Security AuthorizationManager
category: springsecurity
copyright: springsecurity
excerpt: Spring Data JPA
---

## 1. 简介

[Spring Security](https://www.baeldung.com/security-spring)是[Spring Framework](https://www.baeldung.com/spring-tutorial)的一个扩展，它使我们可以轻松地将常见的安全实践构建到我们的应用程序中，这包括用户身份验证和授权、API保护等等。

在本教程中，我们将了解Spring Security中的众多部分之一：AuthorizationManager。我们将了解它如何融入更大的Spring Security生态系统，以及它如何帮助保护我们的应用程序的各种用例。

## 2. 什么是Spring Security AuthorizationManager

**Spring AuthorizationManager是一个接口，它允许我们检查经过身份验证的实体是否有权访问受保护的资源**。Spring Security使用AuthorizationManager实例为基于请求、基于方法和基于消息的组件做出最终的访问控制决策。

作为背景，在了解AuthorizationManager的具体作用之前，Spring Security有几个关键概念有助于理解：

- 实体：可以向系统发出请求的任何事物，例如，可以是人类用户或远程Web服务。
- 身份验证：验证实体是否是其声称的身份的过程，这可以通过用户名/密码、令牌或任何其他方法进行。
- 授权：验证实体是否可以访问资源的过程。
- 资源：系统提供的任何可供访问的信息-例如URL或文档。
- 权限：通常称为角色，这是一个代表实体拥有的权限的逻辑名称，单个实体可能被授予零个或多个权限。

有了这些概念，我们可以更深入地研究AuthorizationManager接口。

### 2.1 如何使用AuthorizationManager

AuthorizationManager是一个简单的接口，仅包含两个方法：

```java
AuthorizationDecision check(Supplier<Authentication> authentication, T object);

void verify(Supplier<Authentication> authentication, T object);
```

这两个方法看起来很相似，因为它们采用相同的参数：

- authentication：提供代表发出请求的实体的身份验证对象的Supplier
- object：所请求的安全对象(将根据请求的性质而有所不同)

但是，每种方法都有不同的用途。第一种方法返回一个AuthorizationDecision，它是boolean值的简单包装，指示实体是否可以访问安全对象。

第二种方法不返回任何内容。**相反，它只是执行授权检查，如果实体无权访问安全对象，则抛出AccessDeniedException**。

### 2.2 Spring Security旧版本

值得注意的是，AuthorizationManager接口是在Spring Security 5.0中引入的，在此接口之前，授权的主要方法是通过AccessDecisionManager接口。**虽然AccessDecisionManager接口在Spring Security的最新版本中仍然存在，但它已被弃用，应避免使用，而应使用AuthorizationManager**。

## 3. AuthorizationManager的实现

Spring提供了AuthorizationManager接口的几种实现，在以下部分中，我们将介绍其中的几种。

### 3.1 AuthenticatedAuthorizationManager

我们将要介绍的第一个实现是AuthenticatedAuthorizationManager，简而言之，**此类仅根据实体是否经过身份验证来返回肯定的授权决策**。此外，它还支持三个级别的身份验证：

- 匿名：该实体未经身份验证
- 记住我：该实体已通过身份验证，并且正在使用[记住的凭证](https://www.baeldung.com/spring-security-remember-me)
- 完全认证：实体已经过认证，并且不使用记住的凭证

请注意，这是[Spring Boot](https://www.baeldung.com/spring-boot)为基于Web的应用程序创建的默认AuthorizationManager。默认情况下，所有端点都将允许访问，无论角色或权限如何，只要它来自经过身份验证的实体。

### 3.2 AuthoritiesAuthorizationManager

此实现与前一个实现类似，不同之处在于**它可以根据多个权限做出决策**。这更适合复杂的应用程序，其中资源可能需要由多个权限访问。

假设有一个博客系统，它使用不同的角色来管理发布过程。创建和保存文章的资源可能可供Author和Editor角色访问。但是，只有Editor角色才能访问发布资源。

### 3.3 AuthorityAuthorizationManager

这个实现相当简单，**它根据实体是否具有特定角色来做出所有授权决策**。

此实现适用于每个资源需要单个角色或权限的简单应用程序。例如，它非常适合保护一组特定的URL，仅允许具有Administrator角色的实体访问。

请注意，此实现将其决策委托给AuthoritiesAuthorizationManager的实例，这也是我们在自定义SecurityFilterChain时调用hasRole()或hasAuthorities()时Spring使用的实现。

### 3.4 RequestMatcherDelegatingAuthorizationManager

此实现实际上并不做出授权决策，相反，**它委托给基于URL模式的另一个实现**，通常是上述管理器类之一。

例如，如果我们有一些公开、可供任何人使用的URL，我们可以将这些URL委托给始终返回肯定授权的无操作实现。然后，我们可以将安全请求委托给负责检查角色的AuthoritiesAuthorizationManager。

事实上，**当我们向SecurityFilterChain添加新的请求匹配器时，Spring正是这么做的**。每次我们配置新的请求匹配器并指定一个或多个所需的角色或权限时，Spring都会创建此类的新实例以及适当的委托。

### 3.5 ObservationAuthorizationManager

我们将要介绍的最后一个实现是ObservationAuthorizationManager，该类实际上只是另一个实现的包装器，并增加了记录与授权决策相关的[指标](https://www.baeldung.com/dropwizard-metrics)的功能。只要应用程序中有有效的ObservationRegistry，Spring就会自动使用此实现。

### 3.6 其他实现

值得一提的是，Spring Security中还有其他几种实现，它们大多数与用于保护方法的各种Spring Security注解有关：

- SecuredAuthorizationManager -> @Secured
- PreAuthorizeAuthorizationManager -> @PreAuthorize
- PostAuthorizeAuthorizationManager -> @PostAuthorize

**本质上，我们可以用来保护资源的任何Spring Security注解都有相应的AuthorityManager实现**。

### 3.7 使用多个AuthorizationManager

在实践中，我们很少只使用AuthorizationManager的单个实例。让我们看一个SecurityFilterChain Bean的示例：

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests((authorize) -> authorize
            .requestMatchers("/posts/publish/**").hasRole("EDITOR")
            .requestMatchers("/posts/create/**").hasAnyRole("EDITOR", "AUTHOR")
            .anyRequest().permitAll());
    return http.build();
}
```

此示例使用了五个不同的AuthorizationManager实例：

- 对hasRole()的调用会创建AuthorityAuthorizationManager的一个实例，该实例又会委托给AuthoritiesAuthorizationManager的一个新实例。
- 对hasAnyRole()的调用也会创建AuthorityAuthorizationManager的实例，该实例又委托给AuthoritiesAuthorizationManager的新实例。
- 对permitAll()的调用使用了Spring Security提供的静态无操作AuthorizationManager，它始终提供积极的授权决策。

具有自身角色的额外请求匹配器以及任何基于方法的注解都将创建附加的AuthorizationManager实例。

## 4. 使用自定义AuthorizationManager

上面提供的实现对于许多应用程序来说已经足够了，但是，与Spring中的许多接口一样，完全可以创建自定义AuthorizationManager来满足我们的任何需求。

让我们定义一个自定义的AuthorizationManager：

```java
AuthorizationManager<RequestAuthorizationContext> customAuthManager() {
    return new AuthorizationManager<RequestAuthorizationContext>() {
        @Override
        public AuthorizationDecision check(Supplier<Authentication> authentication, RequestAuthorizationContext object) {
            // make authorization decision
        }
    };
}
```

然后我们将在自定义SecurityFilterChain时传递这个实例：

```java
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests((authorize) ->
                    authorize.requestMatchers("/custom/**").access(customAuthManager())
    return http.build();
}
```

在本例中，我们使用RequestAuthorizationContext做出授权决策。此类提供对底层HTTP请求的访问，这意味着我们可以根据Cookie、Header等做出决策。我们还可以委托第三方服务、数据库或缓存等结构来做出我们想要的任何类型的授权决策。

## 5. 总结

在本文中，我们仔细研究了Spring Security如何处理授权，我们看到了通用的AuthorizationManager接口以及它的两个方法如何做出授权决策。

我们还看到了该接口的各种实现，以及它们在Spring Security框架中的各个地方的使用方式。

最后，我们创建了一个简单的自定义实现，可用于做出我们的应用程序中所需的任何类型的授权决策。