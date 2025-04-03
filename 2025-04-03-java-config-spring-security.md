---
layout: post
title:  Spring Security的Java配置简介
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

本教程介绍了Spring Security的Java配置，使用户无需使用XML即可轻松配置Spring Security。

Java配置在[Spring 3.1](http://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/new-in-3.1.html)中被添加到Spring框架中，在[Spring 3.2](http://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/new-in-3.2.html)中被扩展到Spring Security，并在用@Configuration标注的类中定义。

## 2. Maven依赖

要将Spring Security集成到Spring Boot应用程序中，我们需要在pom.xml中添加[spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 3. 使用Java配置的Web安全

让我们从Spring Security Java配置的基本示例开始：

```java
@Configuration
public class SecurityConfig {

    @Bean
    public UserDetailsService inMemoryUserDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails user = User.builder()
                .username("user")
                .password(passwordEncoder.encode("password"))
                .roles("USER")
                .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

此配置类使用Spring Security设置基本的内存身份验证机制。

它定义了一个用户名/密码为user/password的用户。此外，它还分配了USER角色。我们使用InMemoryUserDetailsManager，它将用户详细信息存储在内存中。此外，我们需要一个[PasswordEncoder](https://www.baeldung.com/spring-security-5-default-password-encoder) Bean来对用户的密码进行编码：

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```

## 4. Web安全

为了配置我们的授权规则和身份验证机制，我们定义了一个SecurityFilterChain Bean：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests((authz) -> authz
                    .anyRequest().authenticated())
            .httpBasic(withDefaults());
    return http.build();
}
```

上述配置确保对应用程序的任何请求都通过HTTP基本身份验证进行验证。

现在，当我们启动应用程序并导航到http://localhost:8080/app/时，会出现一个基本身份验证登录提示。输入用户名“user”和密码“password”后，服务器将验证凭据。如果身份验证成功，我们就可以访问请求的资源。

## 5. 表单登录

我们可以通过在声明中添加formLogin DSL调用来添加基于浏览器的登录和默认登录页面：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests((authz) -> authz
                    .anyRequest().authenticated())
            .formLogin(withDefaults())
            .httpBasic(withDefaults());
    return http.build();
}
```

在启动时，我们可以看到为我们生成了一个默认页面：

![](/assets/images/2025/springsecurity/javaconfigspringsecurity01.png)

## 6. 角色授权

现在让我们使用角色在每个URL上配置一些简单的授权：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .authorizeHttpRequests((authz) -> authz
                    .requestMatchers("/").hasRole("USER")
                    .requestMatchers("/admin/**").hasRole("ADMIN")
                    .anyRequest().authenticated()
            )
            .formLogin(withDefaults())
            .httpBasic(withDefaults());
    return http.build();
}
```

这些限制了具有USER角色的用户对根(/)的访问以及具有ADMIN角色的用户对/admin/\*\*的访问，而任何经过身份验证的用户都可以访问网站的其余部分。

注意我们如何使用类型安全的API hasRole。

## 7. 注销

与Spring Security的许多其他方面一样，注销具有框架提供的一些很好的默认设置。

默认情况下，注销请求会使会话无效，清除所有身份验证缓存，清除SecurityContextHolder，并重定向到登录页面。

**这是一个简单的注销配置**：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            // ...
            .logout(withDefaults());
    return http.build();
}
```

但是，如果我们想要对可用的处理程序进行更多的控制，那么更完整的实现将是这样的：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http,
                                       LogoutSuccessHandler webSecurityUserLogoutHandler) throws Exception {
    http
            // ...
            .logout((logout) -> logout
                    .logoutSuccessUrl("/")
                    .invalidateHttpSession(true)
                    .logoutSuccessHandler(webSecurityUserLogoutHandler)
                    .deleteCookies("JSESSIONID")
            );
    return http.build();
}

@Bean
public LogoutSuccessHandler webSecurityUserLogoutHandler() {
    return (request, response, authentication) -> {
        System.out.println("User logged out successfully!");
        response.sendRedirect("/app");
    };
}
```

此设置提供了一个干净、安全的注销过程，同时提供了处理注销后操作的灵活性。

## 8. 身份验证

让我们看一下使用Spring Security允许身份验证的另一种方法。

### 8.1 内存身份验证

回想一下，一开始，我们使用内存配置：

```java
@Bean
public UserDetailsService inMemoryUserDetailsService(
        PasswordEncoder passwordEncoder) {
    UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder.encode("password"))
            .roles("USER")
            .build();
    UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder.encode("password"))
            .roles("ADMIN")
            .build();
    return new InMemoryUserDetailsManager(user, admin);
}
```

### 8.2 JDBC身份验证

要将其迁移到JDBC，首先，我们将[H2](https://mvnrepository.com/artifact/com.h2database/h2)依赖添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

另外，我们需要在application.properties中配置H2数据库：

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver
spring.sql.init.schema-locations=classpath:org/springframework/security/core/userdetails/jdbc/users.ddl
```

这定义了我们在下一部分中可以依赖的数据源：

```java
@Bean
public UserDetailsManager jdbcUserDetailsManager(DataSource dataSource,
                                                 PasswordEncoder passwordEncoder) {
    JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);
    UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder.encode("password"))
            .roles("USER")
            .build();
    UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder.encode("password"))
            .roles("ADMIN")
            .build();
    jdbcUserDetailsManager.createUser(user);
    jdbcUserDetailsManager.createUser(admin);
    return jdbcUserDetailsManager;
}
```

另外，我们需要设置一个使用默认用户模式初始化的嵌入式数据源。

当然，对于上述两个示例，我们还需要按照概述定义PasswordEncoder Bean。Spring Security在org/springframework/security/core/userdetails/jdbc/users.ddl中提供了一个。

我们使用[@Profile](https://www.baeldung.com/spring-profiles)(inmemory或jdbc)来选择应该激活哪些UserDetailsManager Bean：

```properties
spring.profiles.active=inmemory
```

## 9. 总结

在本快速教程中，我们介绍了Spring Security的Java配置基础知识，并重点介绍了说明最简单配置场景的代码示例。