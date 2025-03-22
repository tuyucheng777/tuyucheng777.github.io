---
layout: post
title:  如何解决Spring Boot POST请求中的403错误
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

在Web开发中，遇到错误是很常见的事。[HTTP 403](https://www.baeldung.com/spring-security-custom-access-denied-page)禁止错误就是其中一种错误。

在本教程中，我们将学习如何解决Spring Boot [POST请求](https://www.baeldung.com/rest-http-put-vs-post)中的403错误。我们首先了解403错误的含义，然后探索在Spring Boot应用程序中解决该错误的步骤。

## 2. 什么是403错误？

**HTTP 403错误，通常称为“Forbidden”错误，是一种状态码，表示服务器理解了请求，但选择不授权**。这通常意味着客户端缺乏访问请求资源的权限。

值得注意的是，此错误不同于401错误，后者表示服务器需要对客户端进行身份验证，但尚未收到有效的凭据。

## 3. 403错误的原因

有多种因素可能导致Spring Boot应用程序中出现403错误，其中之一是**客户端无法提供[身份验证](https://www.baeldung.com/spring-security-authentication-and-registration)凭据**。在这种情况下，服务器无法验证客户端的权限，因此拒绝请求，从而导致403错误。

另一个可能的原因在于服务器配置。例如，出于安全考虑，**服务器可能配置为拒绝来自某些IP地址或用户代理的请求**。如果请求来自这些被阻止的实体，服务器将响应403错误。

此外，[Spring Security](https://www.baeldung.com/security-spring)默认启用[跨站点请求伪造(CSRF)](https://www.baeldung.com/spring-security-csrf)保护。CSRF是一种诱骗受害者提交恶意请求的攻击，并使用受害者的身份代表他们执行不受欢迎的功能。**如果用于防范此类攻击的CSRF令牌丢失或不正确，服务器也可能会响应错误403**。

## 4. 项目设置

为了演示如何解决403错误，我们将创建一个具有[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)和[spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)依赖项的Spring Boot项目：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

然后我们将创建一个控制器类来处理POST请求：

```java
@PostMapping("/test-request")
public ResponseEntity<String> testPostRequest() {
    return ResponseEntity.ok("POST request successful");
}
```

上述方法带有@PostMapping注解，这意味着它可以处理对服务器的POST请求。成功的POST请求将返回“POST request successful”作为响应。

接下来，我们将通过添加内存用户来配置Spring Security：

```java
@Bean
public InMemoryUserDetailsManager userDetailsService() {
    UserDetails user = User.withUsername("user")
            .password(encoder().encode("userPass"))
            .roles("USER")
            .build();
    return new InMemoryUserDetailsManager(user);
}

@Bean
public PasswordEncoder encoder() {
    return new BCryptPasswordEncoder();
}
```

在上面的代码中，我们将应用程序配置为使用内存中的用户进行请求身份验证，用户的密码使用[BCryptPasswordEncoder](https://www.baeldung.com/spring-security-5-default-password-encoder)进行编码以增强安全性。

最后，我们将配置SecurityFilterChain来接收所有传入的请求：

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeRequests(authorizeRequests -> authorizeRequests.anyRequest()
            .permitAll());
    return http.build();
}
```

在这段代码中，我们将应用程序配置为允许所有传入请求，而无需任何形式的身份验证。

## 5. 解决Spring BootPOST请求中的403错误

在本节中，我们将探讨可能导致403错误的几个因素，并讨论可能的解决方案。

### 5.1 跨站请求伪造(CSRF)保护

默认情况下，Spring Security启用CSRF保护。如果请求标头中缺少CRSF令牌，服务器将响应403错误。此行为并不特定于任何服务器环境，包括本地主机、暂存或生产。

让我们尝试发出POST请求：

```shell
$ curl -X POST -H "Content-Type: application/json" http://localhost:8080/test-request
```

上述请求导致禁止错误：

```json
{"timestamp":"2023-06-24T16:52:05.397+00:00","status":403,"error":"Forbidden","path":"/test-request"}
```

我们可以通过禁用CSRF保护来解决此错误：

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeRequests(authorizeRequests -> authorizeRequests.anyRequest()
                    .permitAll())
            .csrf(AbstractHttpConfigurer::disable);
    return http.build();
}
```

在上面的代码中，我们通过调用disable()方法来禁用CSRF保护。

让我们向”/test-request”端点发出POST请求：

```shell
$ curl -X POST -H "Content-Type: application/json" http://localhost:8080/test-request
```

禁用CRSF后，我们发出POST请求，服务器以预期的HTTP响应“POST request successful.”进行响应。

但是，**需要注意的是，通常不建议在生产应用程序中禁用CRSF保护，CRSF保护是防止跨站点伪造攻击的重要安全措施。因此，建议在状态更改操作的请求标头中包含[CRSF](https://www.baeldung.com/spring-security-csrf#stateless-spring-api)令牌**。

### 5.2 身份验证凭据

**向安全端点提供不正确的身份验证凭据或不提供身份验证凭据可能会导致Spring Boot应用程序出现403错误**。

让我们修改SecurityFilterChain来验证对服务器的所有请求：

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeRequests(authorizeRequests -> authorizeRequests.anyRequest()
                    .authenticated())
            .httpBasic(withDefaults())
            .formLogin(withDefaults())
            .csrf(AbstractHttpConfigurer::disable);
    return http.build();
}
```

在上面的代码中，我们将应用程序配置为在授予访问权限之前对每个请求进行身份验证。如果我们向端点发出POST请求而没有提供正确的身份验证凭据，服务器将响应403错误。

让我们使用创建的内存用户的凭据向”/test-request”端点发出POST请求：

![](/assets/images/2025/springboot/javaspringfix403error01.png)

上图显示，当我们提供正确的身份验证时，服务器以200 OK状态码进行响应。

## 6. 总结

在本文中，我们学习了如何通过禁用CRSF保护并提供正确的身份验证凭据来解决Spring Boot中的403错误。我们还演示了如何配置Spring Security以接受经过身份验证和未经身份验证的请求。此外，我们还重点介绍了Spring Boot应用程序出现403错误的不同原因。

