---
layout: post
title:  OAuth2–@EnableResourceServer与@EnableOAuth2Sso，OAuth2-@EnableResourceServer与@EnableOAuth2Sso
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将讨论Spring Security中的@EnableResourceServer和@EnableOAuth2Sso注解。

我们首先解释**OAuth2客户端和OAuth2资源服务器之间的区别**；然后，我们将讨论一下这些注解可以为我们做什么，并使用[Zuul](https://cloud.spring.io/spring-cloud-netflix/multi/multi__router_and_filter_zuul.html)和简单API的示例演示它们的用法。

出于本文的目的，我们假设你已经具有一些使用Zuul和OAuth2的经验。

## 2. OAuth2客户端和资源服务器

在OAuth2中，有4个不同的[角色](https://tools.ietf.org/html/rfc6749#page-6)我们需要考虑：

- **资源所有者**：能够授予对其受保护资源的访问权限的实体
- **授权服务器**：在成功验证资源所有者并获得其授权后，向客户端授予访问令牌 
- **资源服务器**：需要访问令牌才能允许(或至少考虑)访问其资源的组件
- **客户端**：能够从授权服务器获取访问令牌的实体

使用@EnableResourceServer或@EnableOAuth2Sso标注我们的配置类，指示Spring配置将我们的应用程序转换为上面提到的后两个角色之一的组件。

**@EnableResourceServer注解通过配置[OAuth2AuthenticationProcessingFilter](https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/authentication/OAuth2AuthenticationProcessingFilter.html)和其他同等重要的组件使我们的应用程序能够充当资源服务器**。

查看[ResourceServerSecurityConfigurer](https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/config/annotation/web/configurers/ResourceServerSecurityConfigurer.java)类以更好地了解幕后配置的内容。

相反，**@EnableOAuth2Sso注解将我们的应用程序转换为OAuth2客户端**。它指示Spring配置[OAuth2ClientAuthenticationProcessingFilter](https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/client/filter/OAuth2ClientAuthenticationProcessingFilter.html)以及我们的应用程序需要能够从授权服务器获取访问令牌的其他组件。

查看[SsoSecurityConfigurer](https://github.com/spring-projects/spring-security-oauth2-boot/blob/master/spring-security-oauth2-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/client/SsoSecurityConfigurer.java)类以了解Spring为我们配置的更多详细信息。

将这些注解与一些属性结合起来，使我们能够快速启动和运行，让我们创建两个不同的应用程序来看看它们是如何运作的，以及它们是如何相互补充的：

- 我们的第一个应用程序将是我们的边缘节点，一个简单的Zuul应用程序，它将使用@EnableOAuth2Sso注解。它负责对用户进行身份验证(在授权服务器的帮助下)并将传入的请求委托给其他应用程序。
- 第二个应用程序将使用@EnableResourceServer注解，如果传入请求包含有效的OAuth2访问令牌，则将允许访问受保护的资源。

## 3. Zuul-@EnableOAuth2Sso

让我们首先创建一个Zuul应用程序，它将充当我们的边缘节点，并负责使用OAuth2授权服务器对用户进行身份验证：

```java
@Configuration
@EnableZuulProxy
@EnableOAuth2Sso
@Order(value = 0)
public class AppConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private ResourceServerTokenServices
            resourceServerTokenServices;

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/authorization-server-1/**",
                        "/login").permitAll()
                .anyRequest().authenticated().and()
                .logout().permitAll().logoutSuccessUrl("/");
    }
}
```

使用@EnableOAuth2Sso标注我们的Zuul应用程序还会通知Spring配置[OAuth2TokenRelayFilter](https://javadoc.io/doc/org.springframework.cloud/spring-cloud-security/latest/org/springframework/cloud/security/oauth2/proxy/OAuth2TokenRelayFilter.html)过滤器，此过滤器从用户的HTTP会话中检索先前获取的访问令牌并将其传播到下游。

请注意，我们还在AppConfiguration配置类中使用了[@Order](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/annotation/Order.html)注解，这是为了确保我们的[WebSecurityConfigurerAdapter](https://docs.spring.io/spring-security/site/docs/4.2.5.RELEASE/apidocs/org/springframework/security/config/annotation/web/configuration/WebSecurityConfigurerAdapter.html)创建的过滤器优先于其他WebSecurityConfigurerAdapters创建的过滤器。

例如，我们可以使用@EnableResourceServer标注我们的Zuul应用程序以支持HTTP会话标识符和OAuth2访问令牌。但是，这样做会创建新的过滤器，默认情况下，这些过滤器优先于AppConfiguration类创建的过滤器。发生这种情况的原因是[ResouceServerConfiguration](https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/config/annotation/web/configuration/ResourceServerConfiguration.java)(由[@EnableResourceServer](https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/config/annotation/web/configuration/EnableResourceServer.java)触发的配置类)指定的默认顺序为3，而WebSecurityConfigureAdapter的默认顺序为100。

在转向资源服务器之前，我们需要配置一些属性：

```yaml
zuul:
    routes:
        resource-server-mvc-1: /resource-server-mvc-1/**
        authorization-server-1:
            sensitiveHeaders: Authorization
            path: /authorization-server-1/**
            stripPrefix: false
    add-proxy-headers: true

security:
    basic:
        enabled: false
    oauth2:
        sso:
            loginPath: /login
        client:
            accessTokenUri: http://localhost:8769/authorization-server-1/oauth/token
            userAuthorizationUri: /authorization-server-1/oauth/authorize
            clientId: fooClient
            clientSecret: fooSecret
        resource:
            jwt:
                keyValue: "abc"
            id: fooScope
            serviceId: ${PREFIX:}resource
```

无需过多介绍，使用此配置，我们可以：

- 配置我们的Zuul路由并说明在向下游发送请求之前应该添加/删除哪些标头。
- 为我们的应用程序设置一些OAuth2属性，以便能够与我们的授权服务器通信，并使用对称加密配置[JWT](https://www.baeldung.com/spring-security-oauth-jwt)。

## 4. API–@EnableResourceServer

现在我们已经有了Zuul应用程序，让我们创建资源服务器：

```java
@SpringBootApplication
@EnableResourceServer
@Controller
@RequestMapping("/")
class ResourceServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ResourceServerApplication.class, args);
    }

    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public String helloWorld(Principal principal) {
        return "Hello " + principal.getName();
    }
}
```

这是一个简单的应用程序，它公开一个端点来返回发起请求的主体的名称。

接下来配置一些属性：

```yaml
security:
    basic:
        enabled: false
    oauth2:
        resource:
            jwt:
                keyValue: "abc"
            id: fooScope
            service-id: ${PREFIX:}resource
```

请记住，我们需要一个有效的访问令牌(存储在我们的边缘节点中的用户的HTTP会话中)才能访问我们的资源服务器的端点。

## 5. 总结

在本文中，我们解释了@EnableOAuth2Sso和@EnableResourceServer注解之间的区别，我们还通过Zuul和简单API的实际示例演示了如何使用它们。