---
layout: post
title:  Spring Security中的HttpSecurity与WebSecurity
category: springsecurity
copyright: springsecurity
excerpt: Spring Data JPA
---

## 1. 概述

**[Spring Security](https://www.baeldung.com/security-spring)框架提供了WebSecurity和HttpSecurity类，以提供全局和特定资源的机制来限制对API和资产的访问**。WebSecurity类有助于在全局级别配置安全性，而HttpSecurity提供了为特定资源配置安全性的方法。

在本教程中，我们将详细介绍HttpSecurity和WebSecurity的关键用法。此外，我们还将看到这两个类之间的区别。

## 2. HttpSecurity

[HttpSecurity](https://www.baeldung.com/java-config-spring-security#HTTP=)类有助于为特定的HTTP请求配置安全性。

此外，它允许使用requestMatcher()方法将安全配置限制到特定的HTTP端点。

此外，它还提供了为特定HTTP请求配置授权的灵活性。我们可以使用hasRole()方法创建基于角色的身份验证。

下面是使用HttpSecurity类限制对“/admin/\*\*”的访问的示例代码：

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests((authorize) -> authorize.requestMatchers("/admin/**")
                    .authenticated()
                    .anyRequest()
                    .permitAll())
            .formLogin(withDefaults());
    return http.build();
}
```

在上面的代码中，我们使用HttpSecurity类来限制对“/admin/\*\*”端点的访问，对端点发出的任何请求都需要进行身份验证，然后才能授予访问权限。

此外，HttpSecurity还提供了一种为受限端点配置授权的方法。让我们修改示例代码，以仅允许具有管理员角色的用户访问“/admin/\*\*”端点：

```java
// ...
http.authorizeHttpRequests((authorize) -> authorize.requestMatchers("/admin/**").hasRole("ADMIN")
// ...
```

在这里，我们通过仅允许具有“ADMIN”角色的用户访问端点来为请求提供更多层的安全保护。

此外，HttpSecurity类有助于在Spring Security中配置[CORS](https://www.baeldung.com/spring-cors)和[CSRF](https://www.baeldung.com/spring-security-csrf)保护。

## 3. WebSecurity

WebSecurity类有助于在Spring应用程序中全局配置安全性，我们可以通过公开[WebSecurityCustomizer](https://www.baeldung.com/spring-deprecated-websecurityconfigureradapter) Bean来自定义WebSecurity。

与帮助为特定URL模式或单个资源配置安全规则的HttpSecurity类不同，WebSecurity配置全局应用于所有请求和资源。

此外，它还提供了调试Spring Security过滤器日志记录、忽略某些请求和资源的安全检查或为Spring应用程序配置防火墙的方法。

### 3.1 ignoring()方法

此外，WebSecurity类还提供了一个名为ignoring()的方法。ignoring()方法可帮助Spring Security忽略RequestMatcher的实例，建议注册请求仅针对静态资源。

下面是一个使用ignoring()方法忽略Spring应用程序中的静态资源的示例：

```java
@Bean
WebSecurityCustomizer ignoringCustomizer() {
    return (web) -> web.ignoring().requestMatchers("/resources/**", "/static/**");
}
```

在这里，我们使用ignoring()方法来绕过静态资源的安全检查。

**值得注意的是，Spring建议不要将ignoring()方法用于动态请求，而应仅用于静态资源，因为它会绕过Spring Security过滤器链**。建议将其用于CSS、图像等静态资产。

但是，动态请求需要通过身份验证和授权来提供不同的访问规则，因为它们携带敏感数据。此外，如果我们完全忽略动态端点，我们将失去完全的安全控制，这可能会使应用程序遭受不同的攻击，例如CSRF攻击或SQL注入。

### 3.2 debug()方法

此外，debug()方法可以[记录Spring Security](https://www.baeldung.com/spring-security-enable-logging)内部信息，以协助调试配置或请求失败。这有助于诊断安全规则，而无需调试器。

我们来看一个使用debug()方法调试安全性的示例代码：

```java
@Bean
WebSecurityCustomizer debugSecurity() {
    return (web) -> web.debug(true);
}
```

在这里，我们在WebSecurity实例上调用debug()并将其设置为true，这将全局启用所有安全过滤器的调试日志记录。

### 3.3 httpFirewall()方法

此外，WebSecurity类提供了httpFirewall()方法来为Spring应用程序配置[防火墙](https://www.baeldung.com/spring-security-request-rejected-exception#spring-security-httpfirewall-interface)，它有助于设置规则以在全局级别允许某些操作。

让我们使用httpFirewall()方法来确定在我们的应用程序中应该允许哪些HTTP方法：

```java
@Bean
HttpFirewall allowHttpMethod() {
    List<String> allowedMethods = new ArrayList<String>();
    allowedMethods.add("GET");
    allowedMethods.add("POST");
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowedHttpMethods(allowedMethods);
    return firewall;
}

@Bean
WebSecurityCustomizer fireWall() {
    return (web) -> web.httpFirewall(allowHttpMethod());
}
```

在上面的代码中，我们公开了HttpFirewall Bean来为HTTP方法配置防火墙。默认情况下，允许使用DELETE、GET、HEAD、OPTIONS、PATCH、POST和PUT方法。但是，在我们的示例中，我们仅使用GET和POST方法配置应用程序。

我们创建一个StrictHttpFirewall对象并对其调用setAllowedHttpMethods()方法，该方法接收允许的HTTP方法列表作为参数。

最后，我们通过将allowHttpMethod()方法传递给httpFirewall()方法，公开一个WebSecurityCustomizer Bean来全局配置防火墙。由于防火墙的存在，任何非GET或POST请求都将返回HTTP错误。

## 4. 主要区别

HttpSecurity和WebSecurity配置不会发生冲突，而是可以协同工作以提供全局和特定于资源的安全规则。

但是，如果两者中都配置了类似的安全规则，则WebSecurity配置具有最高优先级：

```java
@Bean
WebSecurityCustomizer ignoringCustomizer() {
    return (web) -> web.ignoring().antMatchers("/admin/**");
}

// ...
 http.authorizeHttpRequests((authorize) -> authorize.antMatchers("/admin/**").hasRole("ADMIN")
// ...
```

在这里，我们在WebSecurity配置中全局忽略“/admin/\*\*”路径，但也在HttpSecurity中配置“/admin/\*\*”路径的访问规则。

在这种情况下，WebSecurity ignoring()配置将覆盖“/admin/\*\*”的HttpSecurity授权。

此外，在SecurityFilterChain中，构建过滤器链时首先执行的是WebSecurity配置，接下来评估的是HttpSecurity规则。

下表显示了HttpSecurity和WebSecurity类之间的主要区别：

| 特征     | WebSecurity                     | HttpSecurity       |
| -------- | ------------------------------ |--------------------|
| 范围     | 全局默认安全规则               | 特定于资源的安全规则         |
| 示例     | 防火墙配置、路径忽略、调试模式 | URL规则、授权、CORS、CSRF |
| 配置方法 | 每个资源的条件配置             | 全局可重用的安全配置         |


## 5. 总结

在本文中，我们通过示例代码了解了HttpSecurity和WebSecurity的关键用法。此外，我们还了解了HttpSecurity如何允许为特定资源配置安全规则，而WebSecurity如何设置全局默认规则。

将它们一起使用可以灵活地在全局和特定资源级别保护Spring应用程序的安全。