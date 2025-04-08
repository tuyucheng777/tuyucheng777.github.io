---
layout: post
title:  使用Spring Security保护Jakarta EE
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本快速教程中，我们将研究如何**使用Spring Security保护Jakarta EE Web应用程序**。

## 2. Maven依赖

让我们从本教程所需的[Spring Security](https://www.baeldung.com/spring-security-with-maven)依赖开始：

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.7.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.7.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>5.7.5</version>
</dependency>
```

最新的Spring Security版本(撰写本教程时)是5.7.5；与往常一样，我们可以检查[Maven Central](https://mvnrepository.com/artifact/org.springframework.security)以获取最新版本。

## 3. 安全配置

接下来，我们需要为现有的Jakarta EE应用程序设置安全配置：

```java
@Configuration
@EnableWebSecurity
public class SpringSecurityConfig {

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.withUsername("user1")
                .password("{noop}user1Pass")
                .roles("USER")
                .build();
        UserDetails admin = User.withUsername("admin")
                .password("{noop}adminPass")
                .roles("ADMIN")
                .build();
        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

为了简单起见，我们实现了一个简单的内存身份验证，用户详细信息是硬编码的。

当不需要完整的持久化机制时，这可用于快速原型设计。

接下来，让我们通过添加SecurityWebApplicationInitializer类将安全性集成到现有系统中：

```java
public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {

    public SecurityWebApplicationInitializer() {
        super(SpringSecurityConfig.class);
    }
}
```

此类将确保在应用程序启动期间加载SpringSecurityConfig。在此阶段，我们已经实现了Spring Security的基本实现。有了这个实现，Spring Security将默认要求对所有请求和路由进行身份验证。

## 4. 配置安全规则

我们可以通过创建SecurityFilterChain Bean进一步定制Spring Security：

```java
 @Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf()
            .disable()
            .authorizeRequests()
            .antMatchers("/auth/login*")
            .anonymous()
            .antMatchers("/home/admin*")
            .hasRole("ADMIN")
            .anyRequest()
            .authenticated()
            .and()
            .formLogin()
            .loginPage("/auth/login")
            .defaultSuccessUrl("/home", true)
            .failureUrl("/auth/login?error=true")
            .and()
            .logout()
            .logoutSuccessUrl("/auth/login");
    return http.build();
}
```

使用antMatchers()方法，我们配置Spring Security以允许匿名访问/auth/login并验证任何其他请求。

### 4.1 自定义登录页面

使用formLogin()方法配置自定义登录页面：

```java
http.formLogin()
    .loginPage("/auth/login")
```

如果未指定，Spring Security会在/login生成默认登录页面：

```html
<html>
<head></head>
<body>
<h1>Login</h1>
<form name='f' action="/auth/login" method='POST'>
    <table>
        <tr>
            <td>User:</td>
            <td><input type='text' name='username' value=''></td>
        </tr>
        <tr>
            <td>Password:</td>
            <td><input type='password' name='password'/></td>
        </tr>
        <tr>
            <td><input name="submit" type="submit"
                       value="submit"/></td>
        </tr>
    </table>
</form>
</body>
</html>
```

### 4.2 自定义登到页面

登录成功后，Spring Security会将用户重定向到应用程序的根目录，我们可以通过指定默认的成功URL来覆盖它：

```java
http.formLogin()
    .defaultSuccessUrl("/home", true)
```

通过将defaultSuccessUrl()方法的alwaysUse参数设置为true，用户将始终被重定向到指定页面。

如果未设置alwaysUse参数或将其设置为false，则用户将被重定向到其在提示进行身份验证之前尝试访问的上一个页面。

类似地，我们也可以指定自定义的失败登到页面：

```java
http.formLogin()
    .failureUrl("/auth/login?error=true")
```

### 4.3 授权

我们可以通过角色限制对资源的访问：

```java
http.formLogin()
    .antMatchers("/home/admin*").hasRole("ADMIN")
```

如果非管理员用户尝试访问/home/admin端点，将收到“访问被拒绝”错误。

我们还可以根据用户的角色限制JSP页面上的数据，这可以使用\<security:authorize\>标签完成：

```html
<security:authorize access="hasRole('ADMIN')">
    This text is only visible to an admin
    <br/>
    <a href="<c:url value="/home/admin" />">Admin Page</a>
    <br/>
</security:authorize>
```

要使用此标签，我们必须在页面顶部包含Spring Security标签库：

```html
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```

## 5. Spring Security XML配置

到目前为止，我们已经了解了如何使用Java配置Spring Security，让我们看一下等效的XML配置。

首先，我们需要在web/WEB-INF/spring文件夹中创建一个包含XML配置的security.xml文件，本文末尾提供了此类security.xml配置文件的示例。

让我们首先配置身份验证管理器和身份验证提供程序，为了简单起见，我们使用简单的硬编码用户凭据：

```xml
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user"
                  password="user123"
                  authorities="ROLE_USER" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

我们刚刚做的是创建一个具有用户名、密码和角色的用户。

或者，我们可以使用密码编码器配置我们的身份验证提供程序：

```xml
<authentication-manager>
    <authentication-provider>
        <password-encoder hash="sha"/>
        <user-service>
            <user name="user"
                  password="d7e6351eaa13189a5a3641bab846c8e8c69ba39f"
                  authorities="ROLE_USER" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

我们还可以指定Spring的UserDetailsService或Datasource的自定义实现作为我们的身份验证提供程序，更多详细信息请参见[此处](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/ns-config.html)。

现在我们已经配置了身份验证管理器，让我们设置安全规则并应用访问控制：

```xml
<http auto-config='true' use-expressions="true">
    <form-login default-target-url="/secure.jsp" />
    <intercept-url pattern="/" access="isAnonymous()" />
    <intercept-url pattern="/index.jsp" access="isAnonymous()" />
    <intercept-url pattern="/secure.jsp" access="hasRole('ROLE_USER')" />
</http>
```

在上面的代码片段中，我们已将HttpSecurity配置为使用表单登录，并将/secure.jsp设置为登录成功URL。我们授予对/index.jsp和“/”路径的匿名访问权限。此外，我们指定对/secure.jsp的访问应需要身份验证，并且经过身份验证的用户应至少具有ROLE_USER级别的权限。

将http标签的auto-config属性设置为true指示Spring Security实现默认行为，我们不必在配置中覆盖这些行为。因此，/login和/logout将分别用于用户登录和注销。还提供了默认登录页面。

我们可以使用自定义登录和注销页面、URL进一步自定义form-login标签，以处理身份验证失败和成功。[Security命名空间附录](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/appendix-namespace.html)列出了form-login(和其他)标签的所有可能属性，某些IDE还可以通过按住ctrl键单击标签来进行检查。

最后，为了在应用程序启动期间加载security.xml配置，我们需要在web.xml中添加以下定义：

```xml
<context-param>                                                                           
    <param-name>contextConfigLocation</param-name>                                        
    <param-value>                                                                         
      /WEB-INF/spring/*.xml                                                             
    </param-value>                                                                        
</context-param>                                                                          
                                                                                          
<filter>                                                                                  
    <filter-name>springSecurityFilterChain</filter-name>                                  
    <filter-class>
      org.springframework.web.filter.DelegatingFilterProxy</filter-class>     
</filter>                                                                                 
                                                                                          
<filter-mapping>                                                                          
    <filter-name>springSecurityFilterChain</filter-name>                                  
    <url-pattern>/*</url-pattern>                                                         
</filter-mapping>                                                                         
                                                                                          
<listener>                                                                                
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

请注意，尝试在同一个Jakarta EE应用程序中同时使用基于XML和Java的配置可能会导致错误。

## 6. 总结

在本文中，我们了解了如何使用Spring Security保护Jakarta EE应用程序，并演示了基于Java和基于XML的配置。

我们还讨论了根据用户角色授予或撤销特定资源访问权限的方法。