---
layout: post
title:  将应用程序从Spring Security 5迁移到Spring Security 6/Spring Boot 3
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

Spring Security 6带来了几个重大变化，包括删除类和弃用的方法，以及引入新的方法。

从Spring Security 5迁移到Spring Security 6可以逐步完成，而不会破坏现有代码库。此外，我们可以使用[OpenRewrite](https://www.baeldung.com/java-openrewrite)等第三方插件来促进迁移到最新版本。

在本教程中，我们将学习如何使用Spring Security 5将现有应用程序迁移到Spring Security 6，我们将替换已弃用的方法并利用[Lambda](https://www.baeldung.com/java-8-Lambda-expressions-tips) DSL简化配置。此外，我们将利用OpenRewrite加快迁移速度。

## 2. Spring Security和Spring Boot版本

[Spring Boot基于Spring框架](https://www.baeldung.com/spring-vs-spring-boot)，Spring Boot的版本均采用Spring框架的最新版本。Spring Boot 2默认采用Spring Security 5，而Spring Boot 3采用Spring Security 6。

要在Spring Boot应用程序中使用Spring Security，我们总是将spring-boot-starter-security依赖添加到pom.xml中。

**但是，我们可以通过在pom.xml的properties部分中指定所需版本来覆盖默认的Spring Security版本**：

```xml
<properties>
    <spring-security.version>5.8.9</spring-security.version>
</properties>
```

在这里，我们指定在项目中使用Spring Security 5.8.9，覆盖默认版本。

值得注意的是，我们还可以通过覆盖properties部分中的默认版本在Spring Boot 2中使用Spring Security 6。

## 3. Spring Security 6中的新功能

Spring Security 6引入了多项功能更新，以提高安全性和稳健性，它现在至少需要Java版本17并使用jakarta命名空间。

**其中一个主要变化是删除了[WebSecurityConfigurerAdapter](https://www.baeldung.com/spring-deprecated-websecurityconfigureradapter)，转而采用基于组件的安全配置**。

此外，authorizeRequests()被删除并被authorizeHttpRequests()取代以定义授权规则。

**此外，它引入了requestMatcher()和securityMatcher()等方法来替代antMatcher()和mvcMatcher()来配置请求资源的安全性**。requestMatcher()方法更安全，因为它会选择合适的RequestMatcher实现来进行请求配置。

其他已弃用的方法(如cors()和csrf())现在有了函数式替代方法。

## 4. 项目设置

首先，让我们通过在pom.xml中添加[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)和[spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)来引导一个Spring Boot 2.7.5项目：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.7.5</version>
</dependency>
```

spring-boot-starter-security依赖使用Spring Security 5。

接下来，让我们创建一个名为WebSecurityConfig的类：

```java
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
class WebSecurityConfig extends WebSecurityConfigurerAdapter {
}
```

在这里，我们用[@EnableWebSecurity](https://www.baeldung.com/spring-enablewebsecurity-vs-enableglobalmethodsecurity)标注该类以启动配置Web请求安全性的过程。此外，我们启用方法级授权。接下来，该类扩展WebSecurityConfigurerAdapter类以提供各种安全配置方法。

此外，让我们定义一个内存用户用于身份验证：

```java
@Override
void configure(AuthenticationManagerBuilder auth) throws Exception {
    UserDetails user = User.withDefaultPasswordEncoder()
            .username("Admin")
            .password("password")
            .roles("ADMIN")
            .build();
    auth.inMemoryAuthentication().withUser(user);
}
```

在上述方法中，我们通过覆盖默认配置来创建一个内存用户。

接下来，让我们通过重写configure(WebSecurity web)方法从安全性中排除静态资源：

```java
@Override
void configure(WebSecurity web) {
    web.ignoring().antMatchers("/js/**", "/css/**");
}
```

最后，让我们通过重写configure(HttpSecurity http)方法来创建HttpSecurity：

```java
@Override
void configure(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .antMatchers("/").permitAll()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .httpBasic();
}
```

值得注意的是，此设置展示了典型的Spring Security 5配置。在后续部分中，我们将此代码迁移到Spring Security 6。

## 5. 将项目迁移到Spring Security 6

Spring建议采用增量迁移方法，以防止在更新到Spring Security 6时破坏现有代码。**在升级到Spring Security 6之前，我们可以先将Spring Boot应用程序升级到Spring Security 5.8.5，并更新代码以使用新功能**。迁移到5.8.5可以让我们为版本6中的预期更改做好准备。

在逐步迁移过程中，我们的IDE可以警告我们已弃用的功能，这有助于增量更新过程。

为简单起见，我们通过将应用程序更新为使用Spring Boot版本3.3.2，将示例项目直接迁移到Spring Security 6。如果应用程序使用Spring Boot版本2，我们可以在properties部分中指定Spring Security 6。

要开始迁移过程，让我们修改pom.xml以使用最新的Spring Boot版本：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>3.3.2</version>
</dependency>
```

在初始设置中，我们使用Spring Boot 2.7.5，其底层使用Spring Security 5。

值得注意的是，Spring Boot 3的最低Java版本是Java 17。

在后续的小节中，我们将重构现有代码以使用Spring Security 6。

### 5.1 @Configuration注解

在Spring Security 6之前，@Configuration注解是@EnableWebSecurity的一部分，但是随着最新更新，我们必须使用@Configuration注解来标注我们的安全配置：

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
}
```

**在这里，我们将@Configuration注解引入到现有代码库中，因为它不再是@EnableWebSecurity注解的一部分**。此外，该注解不再是@EnableMethodSecurity、@EnableWebFluxSecurity或@EnableGlobalMethodSecurity注解的一部分。

**此外，@EnableGlobalMethodSecurity已被标记为弃用，并将由@EnableMethodSecurity取代**。默认情况下，它启用 Spring 的前置注解。因此，我们引入了@EnableMethodSecurity以在方法级别提供授权

### 5.2 WebSecurityConfigurerAdapter

最新更新删除了WebSecurityConfigurerAdapter类并采用基于组件的配置：

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {
}
```

在这里，我们删除了WebSecurityConfigurerAdapter，这样就消除了安全配置的重写方法。**相反，我们可以注册一个用于安全配置的Bean**。我们可以注册WebSecurityCustomizer Bean来配置Web安全，注册SecurityFilterChain Bean来配置HTTP安全，注册InMemoryUserDetails Bean来注册自定义用户等。

### 5.3 WebSecurityCustomizer Bean

让我们通过添加[WebSecurityCustomizer](https://www.baeldung.com/spring-security-httpsecurity-vs-websecurity#websecurity) Bean来修改排除静态资源的方法：

```java
@Bean
WebSecurityCustomizer webSecurityCustomizer() {
    return (web) -> web.ignoring().requestMatchers("/js/**", "/css/**");
}
```

WebSecurityCustomizer接口取代了WebSecurityConfigurerAdapter接口中的configure(WebSecurity web)。

### 5.4 AuthenticationManager Bean

在前面的部分中，我们通过从WebSecurityConfigurerAdapter重写configure(AuthenticationManagerBuilder auth)创建了一个内存用户。

让我们通过注册InMemoryUserDetailsManager Bean来重构身份验证凭证逻辑：

```java
@Bean
InMemoryUserDetailsManager userDetailsService() {
    UserDetails user = User.withDefaultPasswordEncoder()
            .username("Admin")
            .password("admin")
            .roles("USER")
            .build();

    return new InMemoryUserDetailsManager(user);
}
```

在这里，我们定义一个具有USER角色的内存用户来提供基于角色的授权。

### 5.5 HTTP安全配置

在之前的Spring Security版本中，我们通过重写WebSecurityConfigurer类中的configure方法来配置[HttpSecurity](https://www.baeldung.com/spring-security-httpsecurity-vs-websecurity#httpsecurity)。由于它在最新版本中已被删除，因此让我们注册SecurityFilterChain Bean以进行HTTP安全配置：

```java
@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .authorizeHttpRequests(
                    request -> request
                            .requestMatchers("/").permitAll()
                            .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults());
    return http.build();
}
```

在上面的代码中，我们用authorizeHttpRequests()替换了authorizeRequest()方法，新方法使用AuthorizationManager API，简化了重用和自定义。

此外，它还通过延迟身份验证查找来提高性能，仅当请求需要授权时才会进行身份验证查找。

当我们没有自定义规则时，我们使用Customizer.withDefaults()方法来使用默认配置。

此外，我们使用requestMatchers()而不是antMatcher()或mvcMatcher()来保护资源。

### 5.6 RequestCache

请求缓存有助于在需要进行身份验证时保存用户请求，并在用户成功进行身份验证后将用户重定向到请求。在Spring Security 6之前，RequestCache会检查每个传入请求，以查看是否有任何已保存的请求可以重定向到，这会读取每个RequestCache上的HttpSession。

但是，在Spring Security 6中，请求缓存仅检查请求是否包含特殊参数名称“continue”，这样可以提高性能并防止不必要地读取HttpSession。

## 6. 使用OpenRewrite

此外，我们可以使用OpenRewrite等第三方工具将现有的Spring Boot应用程序迁移到Spring Boot 3。由于Spring Boot 3使用Spring Security 6，因此它还将安全配置迁移到版本6。

要使用OpenRewrite，我们可以向pom.xml添加一个[插件](https://mvnrepository.com/artifact/org.openrewrite.maven/rewrite-maven-plugin)：

```xml
<plugin>
    <groupId>org.openrewrite.maven</groupId>
    <artifactId>rewrite-maven-plugin</artifactId>
    <version>5.23.1</version>
    <configuration>
        <activeRecipes>
            <recipe>org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_0</recipe>
        </activeRecipes>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.openrewrite.recipe</groupId>
            <artifactId>rewrite-spring</artifactId>
            <version>5.5.0</version>
        </dependency>
    </dependencies>
</plugin>
```

这里我们通过recipe属性指定升级到Spring Boot版本3，OpenRewrite有很多可用于升级Java项目的recipe可供选择。

最后，我们运行迁移命令：

```shell
$ mvn rewrite:run
```

上述命令将项目迁移到Spring Boot 3，包括安全配置。但是，OpenRewrite目前不使用Lambda DSL进行迁移的安全配置。当然，这在未来的版本中可能会发生变化。

## 7. 总结

在本文中，我们看到了通过替换弃用的类和方法将现有代码库从Spring Security 5迁移到Spring Security 6的分步指南。此外，我们还了解了如何使用第三方插件来自动执行迁移。