---
layout: post
title:  OAuth 2.0和动态客户端注册(使用Spring Security OAuth遗留堆栈)
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 简介

在本教程中，我们将使用OAuth 2.0准备动态客户端注册。OAuth 2.0是一个授权框架，可用于获取HTTP服务上用户帐户的有限访问权限。OAuth 2.0客户端是想要访问用户帐户的应用程序，此客户端可以是外部Web应用程序、用户代理或只是本机客户端。

为了实现动态客户端注册，我们将把凭证存储在数据库中，而不是硬编码配置。我们要扩展的应用程序最初在[Spring REST API + OAuth2教程](https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)中描述。

注意：本文使用的是[Spring OAuth遗留项目](https://spring.io/projects/spring-authorization-server)。

## 2. Maven依赖

我们首先设置以下一组依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>    
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
</dependency>
```

请注意，我们使用spring-jdbc，因为我们将使用DB来存储新注册的用户和密码。

## 3. OAuth 2.0服务器配置

首先，我们需要配置我们的OAuth 2.0授权服务器，主要配置在以下类中：

```java
@Configuration
@PropertySource({ "classpath:persistence.properties" })
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    // config
}
```

我们需要配置一些主要的东西；让我们从ClientDetailsServiceConfigurer开始：

```java
@Override
public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
    clients.jdbc(dataSource());

    // ...		
}
```

**这将确保我们使用持久层来获取客户端信息**。

当然，我们要设置这个标准数据源：

```java
@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();

    dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
    dataSource.setUrl(env.getProperty("jdbc.url"));
    dataSource.setUsername(env.getProperty("jdbc.user"));
    dataSource.setPassword(env.getProperty("jdbc.pass"));
    return dataSource;
}
```

因此，现在我们的应用程序将使用数据库作为注册客户端的来源，而不是典型的硬编码内存客户端。

## 4. DB模式

现在让我们定义用于存储OAuth客户端的SQL结构：

```sql
create table oauth_client_details (
    client_id VARCHAR(256) PRIMARY KEY,
    resource_ids VARCHAR(256),
    client_secret VARCHAR(256),
    scope VARCHAR(256),
    authorized_grant_types VARCHAR(256),
    web_server_redirect_uri VARCHAR(256),
    authorities VARCHAR(256),
    access_token_validity INTEGER,
    refresh_token_validity INTEGER,
    additional_information VARCHAR(4096),
    autoapprove VARCHAR(256)
);
```

我们应该关注oauth_client_details中最重要的字段是：

- client_id：存储新注册客户端的ID
- client_secret：存储客户端的密码
- access_token_validity：指示客户端是否仍然有效
- authorities：指示特定客户端允许哪些角色
- scope：允许的操作，例如在Facebook上写状态等
- authorized_grant_types，提供有关用户如何登录特定客户端的信息(在我们的示例中，它是使用密码的表单登录)

请注意，每个客户端与用户都有一对多的关系，这自然意味着多个用户可以使用单个客户端。

## 5. 保存一些客户端

通过SQL模式定义，我们最终可以在系统中创建一些数据-并基本上定义一个客户端。

我们将使用以下data.sql脚本(Spring Boot将默认运行)来初始化数据库：

```sql
INSERT INTO oauth_client_details
	(client_id, client_secret, scope, authorized_grant_types,
	web_server_redirect_uri, authorities, access_token_validity,
	refresh_token_validity, additional_information, autoapprove)
VALUES
	("fooClientIdPassword", "secret", "foo,read,write,
	"password,authorization_code,refresh_token", null, null, 36000, 36000, null, true);
```

上一节提供了oauth_client_details中最重要的字段的描述。

## 6. 测试

为了测试动态客户端注册，我们需要分别在8081和8082端口上运行spring-security-oauth-server和spring-security-oauth-resource项目。

现在，我们可以编写一些实时测试了。

假设我们注册了id为fooClientIdPassword的客户端，并且该客户端有读取foos的权限。

首先，我们将尝试使用已定义的客户端从授权服务器获取访问令牌：

```java
@Test
public void givenDBUser_whenRevokeToken_thenAuthorized() {
    String accessToken = obtainAccessToken("fooClientIdPassword", "john", "123");

    assertNotNull(accessToken);
}
```

获取访问令牌的逻辑如下：

```java
private String obtainAccessToken(String clientId, String username, String password) {
    Map<String, String> params = new HashMap<String, String>();
    params.put("grant_type", "password");
    params.put("client_id", clientId);
    params.put("username", username);
    params.put("password", password);
    Response response = RestAssured.given().auth().preemptive()
        .basic(clientId, "secret").and().with().params(params).when()
        .post("http://localhost:8081/spring-security-oauth-server/oauth/token");
    return response.jsonPath().getString("access_token");
}
```

## 7. 总结

在本教程中，我们学习了如何使用OAuth 2.0框架动态注册无限数量的客户端。