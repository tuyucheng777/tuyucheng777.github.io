---
layout: post
title:  使用Spring Security实现REST服务的基本身份验证和摘要身份验证
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

本文讨论如何**在REST API的同一URI结构上设置[基本身份验证和摘要身份验证](https://www.baeldung.com/cs/digest-vs-basic-authentication)**。在上一篇文章中，我们讨论了保护REST服务的另一种方法-[基于表单的身份验证](https://www.baeldung.com/securing-a-restful-web-service-with-spring-security)，因此基本身份验证和摘要身份验证是自然的替代方案，也是更RESTful的替代方案。

## 2. 基本认证配置

基于表单的身份验证对于RESTful服务来说并不理想，主要原因是Spring Security将使用会话-这当然是服务器上的状态，因此REST中的无状态约束实际上被忽略了。

我们首先设置基本身份验证-首先我们从主<http\>安全元素中删除旧的自定义入口点和过滤器：

```xml
<http create-session="stateless">
    <intercept-url pattern="/api/admin/**" access="ROLE_ADMIN" />

    <http-basic />
</http>
```

请注意，使用一行配置<http-basic/\>来添加对基本身份验证的支持，该配置行负责处理BasicAuthenticationFilter和BasicAuthenticationEntryPoint的创建和连接。

### 2.1 满足无状态约束-摆脱会话

RESTful架构风格的主要约束之一是客户端-服务器通信完全无状态，正如[原始论文](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)所述：

> 5.1.3 无状态
>
> 接下来，我们为客户端-服务器交互添加一个约束：通信本质上必须是无状态的，就像第3.4.3节(图5-3)中的客户端-无状态-服务器(CSS)风格一样，这样从客户端到服务器的每个请求都必须包含理解请求所需的所有信息，并且不能利用服务器上存储的任何上下文。因此，会话状态完全保存在客户端上。

服务器上的Session概念在Spring Security中有着悠久的历史，到目前为止，完全删除它仍然很困难，尤其是在使用命名空间进行配置时。

但是，Spring Security为命名空间配置添加了一个用于会话创建的stateless[选项](https://github.com/spring-projects/spring-security/issues/1667)，这有效地保证了Spring不会创建或使用任何会话。这个新选项的作用是从安全过滤器链中彻底删除所有与会话相关的过滤器，确保对每个请求都执行身份验证。

## 3. 摘要认证的配置

从前面的配置开始，设置摘要身份验证所需的过滤器和入口点将被定义为Bean。然后，摘要入口点将在后台覆盖由<http-basic\>创建的入口点。最后，将使用安全命名空间的after语义将自定义摘要过滤器引入安全过滤器链中，以将其直接定位在基本身份验证过滤器之后。

```xml
<http create-session="stateless" entry-point-ref="digestEntryPoint">
   <intercept-url pattern="/api/admin/**" access="ROLE_ADMIN" />

   <http-basic />
   <custom-filter ref="digestFilter" after="BASIC_AUTH_FILTER" />
</http>

<beans:bean id="digestFilter" class="org.springframework.security.web.authentication.www.DigestAuthenticationFilter">
   <beans:property name="userDetailsService" ref="userService" />
   <beans:property name="authenticationEntryPoint" ref="digestEntryPoint" />
</beans:bean>

<beans:bean id="digestEntryPoint" class="org.springframework.security.web.authentication.www.DigestAuthenticationEntryPoint">
   <beans:property name="realmName" value="Contacts Realm via Digest Authentication"/>
   <beans:property name="key" value="acegi" />
</beans:bean>

<authentication-manager>
   <authentication-provider>
      <user-service id="userService">
         <user name="eparaschiv" password="eparaschiv" authorities="ROLE_ADMIN" />
         <user name="user" password="user" authorities="ROLE_USER" />
      </user-service>
   </authentication-provider>
</authentication-manager>
```

不幸的是，安全命名空间不[支持](https://github.com/spring-projects/spring-security/issues/2095)自动配置摘要式身份验证，而基本身份验证可以通过<http-basic\>配置。因此，必须手动定义必要的Bean并将其注入到安全配置中。

## 4. 在同一个RESTful服务中支持两种身份验证协议

单独的基本或摘要式身份验证可以在Spring Security中轻松实现；它对同一个RESTful Web服务、同一个URI映射都支持这两种身份验证，这为服务的配置和测试带来了新的复杂性。

### 4.1 匿名请求

在安全链中同时使用基本过滤器和摘要过滤器的情况下，Spring Security处理匿名请求(不包含身份验证凭据(Authorization HTTP标头)的方法是：两个身份验证过滤器将找不到凭据，并将继续执行过滤器链。然后，看到请求未通过身份验证，就会抛出AccessDeniedException并在ExceptionTranslationFilter中捕获，从而启动摘要入口点，提示客户端输入凭据。

基本过滤器和摘要过滤器的职责都很狭窄-如果它们无法识别请求中的身份验证凭据类型，它们将继续执行安全过滤器链。正因为如此，Spring Security可以灵活地配置为在同一URI上支持多种身份验证协议。

当请求包含正确的身份验证凭据(基本或摘要)时，将正确使用该协议。但是，对于匿名请求，客户端只会被提示输入摘要身份验证凭据。这是因为摘要入口点被配置为Spring Security链的主要和唯一入口点；因此摘要身份验证可以被视为默认的。

### 4.2 带有身份验证凭据的请求

具有基本身份验证凭据的请求将通过以前缀“Basic”开头的授权标头来识别，处理此类请求时，凭据将在基本身份验证过滤器中解码，并授权该请求。同样，具有摘要身份验证凭据的请求将使用前缀“Digest”作为其授权标头。

## 5. 测试两种场景

测试将在使用基本或摘要进行身份验证后创建新资源来使用REST服务：

```java
@Test
public void givenAuthenticatedByBasicAuth_whenAResourceIsCreated_then201IsReceived(){
    // Given
    // When
    Response response = given()
            .auth().preemptive().basic( ADMIN_USERNAME, ADMIN_PASSWORD )
            .contentType( HttpConstants.MIME_JSON ).body( new Foo( randomAlphabetic( 6 ) ) )
            .post( paths.getFooURL() );

    // Then
    assertThat( response.getStatusCode(), is( 201 ) );
}

@Test
public void givenAuthenticatedByDigestAuth_whenAResourceIsCreated_then201IsReceived(){
    // Given
    // When
    Response response = given()
            .auth().digest( ADMIN_USERNAME, ADMIN_PASSWORD )
            .contentType( HttpConstants.MIME_JSON ).body( new Foo( randomAlphabetic( 6 ) ) )
            .post( paths.getFooURL() );

    // Then
    assertThat( response.getStatusCode(), is( 201 ) );
}
```

请注意，使用基本身份验证的测试会预先将凭据添加到请求中，无论服务器是否已质询身份验证。这是为了确保服务器不需要质询客户端的凭据，因为如果服务器这样做，则质询将是摘要凭据，因为这是默认设置。

## 6. 总结

本文介绍了RESTful服务的基本身份验证和摘要身份验证的配置和实现，主要使用Spring Security命名空间支持以及框架中的一些新功能。