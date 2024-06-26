---
layout: post
title:  Spring Security中的AuthenticationManagerResolver指南
category: springreactive
copyright: springreactive
excerpt: Spring Security
---

## 1. 简介

在本教程中，我们介绍了AuthenticationManagerResolver，然后展示了如何将其用于Basic和OAuth2身份验证流程。

## 2. AuthenticationManager是什么？

简单地说，[AuthenticationManager](https://spring.io/guides/topicals/spring-security-architecture)是身份验证的主要策略接口。

如果输入身份验证的主体有效且经过验证，则AuthenticationManager#authenticate返回一个Authentication实例，其中authenticated标志设置为true。否则，如果委托人无效，它将抛出AuthenticationException。对于最后一种情况，如果无法决定则返回null。

ProviderManager是AuthenticationManager的默认实现。它将身份验证过程委托给AuthenticationProvider实例列表。

如果我们创建一个[SecurityFilterChain](https://www.baeldung.com/spring-security-multiple-auth-providers) bean，我们可以设置全局或本地AuthenticationManager。对于本地AuthenticationManager，我们可以创建一个AuthenticationManager bean，通过HttpSecurity访问AuthenticationManagerBuilder。

[AuthenticationManagerBuilder](https://www.baeldung.com/java-config-spring-security)是一个辅助类，它简化了UserDetailService、AuthenticationProvider和其他依赖项的设置，以构建AuthenticationManager。

对于全局AuthenticationManager，我们应该将AuthenticationManager定义为一个bean。

## 3. 为什么使用AuthenticationManagerResolver？

AuthenticationManagerResolver让Spring为每个上下文选择一个AuthenticationManager。这是Spring Security 5.2.0版本中添加的[新功能](https://github.com/spring-projects/spring-security/issues/6722)：

```java
public interface AuthenticationManagerResolver<C> {
    AuthenticationManager resolve(C context);
}
```

AuthenticationManagerResolver#resolve可以返回基于通用上下文的AuthenticationManager实例。换句话说，如果我们想根据它来解析AuthenticationManager，我们可以将一个类设置为上下文。

Spring Security已将AuthenticationManagerResolver集成到身份验证流程中，并将HttpServletRequest和ServerWebExchange作为上下文。

## 4. 使用场景

让我们看看如何在实践中使用AuthenticationManagerResolver。

例如，假设一个系统有两组用户：员工和客户。这两个组具有特定的身份验证逻辑并具有单独的数据存储。此外，这些组中的任何一个的用户都只允许调用他们的相关URL。

## 5. AuthenticationManagerResolver是如何工作的？

我们可以在任何需要动态选择AuthenticationManager的地方使用AuthenticationManagerResolver，但在本教程中，我们有兴趣在内置身份验证流程中使用它。

首先，让我们设置一个AuthenticationManagerResolver，然后将其用于Basic和OAuth2身份验证。

### 5.1 设置AuthenticationManagerResolver

让我们从为安全配置创建一个类开始。

```java
@Configuration
public class CustomWebSecurityConfigurer {
    // ...
}
```

然后，让我们添加一个为客户返回AuthenticationManager的方法：

```java
AuthenticationManager customersAuthenticationManager() {
    return authentication -> {
        if (isCustomer(authentication)) {
            return new UsernamePasswordAuthenticationToken(/*credentials*/);
        }
        throw new UsernameNotFoundException(/*principal name*/);
    };
}
```

员工的AuthenticationManager在逻辑上是相同的，只是我们将isCustomer替换为isEmployee：

```java
public AuthenticationManager employeesAuthenticationManager() {
    return authentication -> {
        if (isEmployee(authentication)) {
            return new UsernamePasswordAuthenticationToken(/*credentials*/);
        }
        throw new UsernameNotFoundException(/*principal name*/);
    };
}
```

最后，让我们添加一个根据请求的URL解析的AuthenticationManagerResolver：

```java
AuthenticationManagerResolver<HttpServletRequest> resolver() {
    return request -> {
        if (request.getPathInfo().startsWith("/employee")) {
            return employeesAuthenticationManager();
        }
        return customersAuthenticationManager();
    };
}
```

### 5.2 对于基本身份验证

我们可以使用AuthenticationFilter来动态解析每个请求的AuthenticationManager。AuthenticationFilter在5.2版本中被添加到Spring Security。

如果我们将它添加到我们的安全过滤器链中，那么对于每个匹配的请求，它首先检查它是否可以提取任何身份验证对象。如果是，则它向AuthenticationManagerResolver询问合适的AuthenticationManager并继续流程。

首先，让我们在CustomWebSecurityConfigurer中添加一个方法来创建AuthenticationFilter：

```java
private AuthenticationFilter authenticationFilter() {
    AuthenticationFilter filter = new AuthenticationFilter(resolver(), authenticationConverter());
    filter.setSuccessHandler((request, response, auth) -> {});
    return filter;
}
```

将AuthenticationFilter#successHandler设置为无操作SuccessHandler的原因是为了防止成功验证后重定向的默认行为。

然后，我们可以通过在我们的CustomWebSecurityConfigurer中创建一个SecurityFilterChain bean来将这个过滤器添加到我们的安全过滤器链中：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.addFilterBefore(authenticationFilter(), BasicAuthenticationFilter.class);
    return http.build();
}
```

### 5.3 对于OAuth2身份验证

BearerTokenAuthenticationFilter负责OAuth2身份验证。BearerTokenAuthenticationFilter#doFilterInternal方法检查请求中的BearerTokenAuthenticationToken，如果可用，则解析适当的AuthenticationManager以验证令牌。

OAuth2ResourceServerConfigurer用于设置BearerTokenAuthenticationFilter。

因此，我们可以通过创建SecurityFilterChain bean在CustomWebSecurityConfigurer中为我们的资源服务器设置AuthenticationManagerResolver：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.oauth2ResourceServer()
      	.authenticationManagerResolver(resolver());
    return http.build();
}
```

## 6. Reactive Applications中解析AuthenticationManager

对于响应式Web应用程序，我们仍然可以从根据上下文解析AuthenticationManager的概念中受益。但在这里我们有ReactiveAuthenticationManagerResolver：

```java
@FunctionalInterface
public interface ReactiveAuthenticationManagerResolver<C> {
    Mono<ReactiveAuthenticationManager> resolve(C context);
}
```

它返回ReactiveAuthenticationManager的Mono。ReactiveAuthenticationManager是AuthenticationManager的响应式等价物，因此它的authenticate方法返回Mono。

### 6.1 设置ReactiveAuthenticationManagerResolver

让我们从为安全配置创建一个类开始：

```java
@EnableWebFluxSecurity
@EnableReactiveMethodSecurity
public class CustomWebSecurityConfig {
    // ...
}
```

接下来，让我们在这个类中为客户定义ReactiveAuthenticationManager ：

```java
ReactiveAuthenticationManager customersAuthenticationManager() {
    return authentication -> customer(authentication)
        .switchIfEmpty(Mono.error(new UsernameNotFoundException(/*principal name*/)))
        .map(b -> new UsernamePasswordAuthenticationToken(/*credentials*/));
}
```

之后，我们将为员工定义ReactiveAuthenticationManager：

```java
public ReactiveAuthenticationManager employeesAuthenticationManager() {
    return authentication -> employee(authentication)
        .switchIfEmpty(Mono.error(new UsernameNotFoundException(/*principal name*/)))
        .map(b -> new UsernamePasswordAuthenticationToken(/*credentials*/));
}
```

最后，我们根据我们的场景设置了一个ReactiveAuthenticationManagerResolver：

```java
ReactiveAuthenticationManagerResolver<ServerWebExchange> resolver() {
    return exchange -> {
        if (match(exchange.getRequest(), "/employee")) {
            return Mono.just(employeesAuthenticationManager());
        }
        return Mono.just(customersAuthenticationManager());
    };
}
```

### 6.2 对于基本身份验证

在响应式Web应用程序中，我们可以使用AuthenticationWebFilter进行身份验证。它验证请求并填充安全上下文。

AuthenticationWebFilter首先检查请求是否匹配。之后，如果请求中有身份验证对象，它会从ReactiveAuthenticationManagerResolver获取适合请求的ReactiveAuthenticationManager并继续身份验证流程。

因此，我们可以在安全配置中设置自定义的AuthenticationWebFilter：

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    return http
        .authorizeExchange()
        .pathMatchers("/**")
        .authenticated()
        .and()
        .httpBasic()
        .disable()
        .addFilterAfter(
          	new AuthenticationWebFilter(resolver()), 
          	SecurityWebFiltersOrder.REACTOR_CONTEXT
        )
        .build();
}
```

首先，我们禁用ServerHttpSecurity#httpBasic以阻止正常的身份验证流程，然后用AuthenticationWebFilter手动替换它，传入我们的自定义解析器。

### 6.3 对于OAuth2身份验证

我们可以使用ServerHttpSecurity#oauth2ResourceServer配置ReactiveAuthenticationManagerResolver。ServerHttpSecurity#build添加一个AuthenticationWebFilter实例和我们的解析器到安全过滤器链中。

因此，让我们在安全配置中为OAuth2身份验证过滤器设置AuthenticationManagerResolver：

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    return http
        // ...
        .and()
        .oauth2ResourceServer()
        .authenticationManagerResolver(resolver())
        .and()
        // ...;
}
```

## 7. 总结

在本文中，我们在一个简单的场景中使用了AuthenticationManagerResolver进行基本和OAuth2身份验证。

而且，我们还探索了ReactiveAuthenticationManagerResolver在响应式Spring Web应用程序中用于基本和OAuth2身份验证的用法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-reactive-modules/spring-5-reactive-security)上获得。