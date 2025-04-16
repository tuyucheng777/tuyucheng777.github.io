---
layout: post
title:  Cloud Foundry UAA快速指南
category: security
copyright: security
excerpt: Cloud Foundry
---

## 1. 概述

**Cloud Foundry用户帐户和身份验证(CF UAA)是一种身份管理和授权服务**；更准确地说，它是一个OAuth 2.0提供程序，允许对客户端应用程序进行身份验证和颁发令牌。

在本教程中，我们将介绍设置CF UAA服务器的基础知识。然后，我们将了解如何使用它来保护资源服务器应用程序。

但在此之前，让我们先明确一下UAA在[OAuth 2.0](https://auth0.com/docs/protocols/oauth2)授权框架中的作用。

## 2. Cloud Foundry UAA和OAuth 2.0

让我们首先了解UAA与OAuth 2.0规范的关系。

OAuth 2.0规范定义了四个可以相互连接的[参与者](https://tools.ietf.org/html/rfc6749#section-1.1)：资源所有者、资源服务器、客户端和授权服务器。

作为OAuth 2.0提供商，UAA扮演授权服务器的角色。这意味着**它的主要目标是为客户端应用程序颁发访问令牌，并为资源服务器验证这些令牌**。

为了允许这些参与者进行交互，我们首先需要设置一个UAA服务器，然后再实现两个应用程序：一个作为客户端，另一个作为资源服务器。

我们将在客户端使用[authorization_code授权](https://tools.ietf.org/html/rfc6749#section-1.3.1)流程，我们将在资源服务器中使用Bearer token授权。为了更安全、更高效的握手，我们将使用签名的JWT作为[访问令牌](https://tools.ietf.org/html/rfc6749#section-1.4)。

## 3. 设置UAA服务器

首先，**我们将安装UAA并填充一些演示数据**。

安装完成后，我们将注册一个名为webappclient的客户端应用程序。然后，我们将创建一个名为appuser的用户，该用户具有两个角色：resource.read和resource.write。

### 3.1 安装

UAA是一个Java Web应用程序，可以在任何兼容的Servlet容器中运行，在本教程中，我们将[使用Tomcat](http://tomcat.apache.org/whichversion.html)。

让我们**下载[UAA war](https://mvnrepository.com/artifact/org.cloudfoundry.identity/cloudfoundry-identity-uaa)并将其存入我们的Tomcat部署中**：

```shell
wget -O $CATALINA_HOME/webapps/uaa.war \
  https://search.maven.org/remotecontent?filepath=org/cloudfoundry/identity/cloudfoundry-identity-uaa/4.27.0/cloudfoundry-identity-uaa-4.27.0.war
```

不过，在启动它之前，我们需要配置它的数据源和JWS密钥对。

### 3.2 所需配置

**默认情况下，UAA从其类路径上的uaa.yml读取配置**。但是，由于我们刚刚下载了war文件，因此最好告诉UAA文件系统上的自定义位置。

我们可以通过**设置UAA_CONFIG_PATH属性**来实现这一点：

```shell
export UAA_CONFIG_PATH=~/.uaa
```

或者，我们可以设置CLOUD_FOUNDRY_CONFIG_PATH。或者，我们可以使用UAA_CONFIG_URL指定远程位置。

然后，我们可以将[UAA所需的配置](https://raw.githubusercontent.com/cloudfoundry/uaa/4.27.0/uaa/src/main/resources/required_configuration.yml)复制到我们的配置路径中：

```shell
wget -qO- https://raw.githubusercontent.com/cloudfoundry/uaa/4.27.0/uaa/src/main/resources/required_configuration.yml \
  > $UAA_CONFIG_PATH/uaa.yml
```

请注意，我们删除最后三行，因为我们马上要替换它们。

### 3.3 配置数据源

因此，让我们配置数据源，**UAA将在其中存储有关客户端的信息**。

出于本教程的目的，我们将使用HSQLDB：

```shell
export SPRING_PROFILES="default,hsqldb"
```

当然，由于这是一个Spring Boot应用程序，我们也可以在uaa.yml中将其指定为spring.profiles属性。

### 3.4 配置JWS密钥对

由于我们使用JWT，**UAA需要有一个私钥来签署UAA颁发的每个JWT**。

OpenSSL使这变得简单：

```shell
openssl genrsa -out signingkey.pem 2048
openssl rsa -in signingkey.pem -pubout -out verificationkey.pem
```

授权服务器将使用私钥对JWT进行签名，我们的客户端和资源服务器将使用公钥验证该签名。

我们将它们导出到JWT_TOKEN_SIGNING_KEY和JWT_TOKEN_VERIFICATION_KEY：

```shell
export JWT_TOKEN_SIGNING_KEY=$(cat signingkey.pem)
export JWT_TOKEN_VERIFICATION_KEY=$(cat verificationkey.pem)
```

同样，我们可以通过jwt.token.signing-key和jwt.token.verification-key属性在uaa.yml中指定这些。

### 3.5 启动UAA

最后，我们启动UAA：

```shell
$CATALINA_HOME/bin/catalina.sh run
```

此时，我们应该有一个可用的UAA服务器，网址为[http://localhost:8080/uaa](http://localhost:8080/uaa)。

如果我们访问[http://localhost:8080/uaa/info](http://localhost:8080/uaa/info)，我们将看到一些基本的启动信息

### 3.6 安装UAA命令行客户端

**CF UAA命令行客户端是管理UAA的主要工具**，但要使用它，我们需要先[安装Ruby](https://rubygems.org/)：

```shell
sudo apt install rubygems
gem install cf-uaac
```

然后，我们可以配置uaac以指向我们正在运行的UAA实例：

```shell
uaac target http://localhost:8080/uaa
```

请注意，如果我们不想使用命令行客户端，也可以使用UAA的HTTP客户端。

### 3.7 使用UAAC填充客户端和用户

现在我们已经安装了uaac，让我们用一些演示数据填充UAA。**至少，我们需要：一个客户端、一个用户以及resource.read和resource.write组**。

因此，要进行任何管理，我们都需要自行进行身份验证。我们将选择UAA附带的默认管理员，**该管理员具有创建其他客户端、用户和组的权限**：

```shell
uaac token client get admin -s adminsecret
```

(当然，在发货前我们肯定需要通过[oauth-clients.xml](https://github.com/cloudfoundry/uaa/blob/master/uaa/src/main/webapp/WEB-INF/spring/oauth-clients.xml)文件更改这个帐户。)

基本上，我们可以将此命令理解为：“给我一个token，使用client凭据，其中client_id为admin，密钥为adminsecret”。

如果一切顺利，我们将看到一条成功消息：

```text
Successfully fetched token via client credentials grant.
```

该令牌存储在uaac的状态中。

**现在，以admin身份操作，我们可以使用client add注册一个名为webappclient的客户端**：

```shell
uaac client add webappclient -s webappclientsecret \ 
--name WebAppClient \ 
--scope resource.read,resource.write,openid,profile,email,address,phone \ 
--authorized_grant_types authorization_code,refresh_token,client_credentials,password \ 
--authorities uaa.resource \ 
--redirect_uri http://localhost:8081/login/oauth2/code/uaa
```

**另外，我们可以使用user add注册一个名为appuser的用户**：

```shell
uaac user add appuser -p appusersecret --emails appuser@acme.com
```

接下来，我们将使用group add添加两个组-resource.read和resource.write：

```shell
uaac group add resource.read
uaac group add resource.write
```

最后，我们通过member add将这些组分配给appuser：

```shell
uaac member add resource.read appuser
uaac member add resource.write appuser
```

到目前为止，我们所做的是：

- 安装并配置UAA
- 安装uaac
- 添加示例客户端、用户和组
- 
## 4. OAuth 2.0客户端

在本节中，我们将**使用Spring Boot创建一个OAuth 2.0客户端应用程序**。

### 4.1 应用程序设置

让我们首先访问[Spring Initializr](https://start.spring.io/)并生成一个Spring Boot Web应用程序，我们仅选择Web和OAuth2客户端组件：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

在此示例中，我们使用了[Spring Boot 3.1.5版本](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-parent)。

接下来，**我们需要注册我们的客户端webappclient**。

很简单，我们需要为应用程序提供client-id、client-secret和UAA的issuer-uri，我们还将指定此客户端希望用户授予它的OAuth 2.0范围：

```properties
#registration
spring.security.oauth2.client.registration.uaa.client-id=webappclient
spring.security.oauth2.client.registration.uaa.client-secret=webappclientsecret
spring.security.oauth2.client.registration.uaa.scope=resource.read,resource.write,openid,profile

#provider
spring.security.oauth2.client.provider.uaa.issuer-uri=http://localhost:8080/uaa/oauth/token
```

有关这些属性的更多信息，我们可以查看[注册](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/security/oauth2/client/OAuth2ClientProperties.Registration.html)和[提供程序](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/security/oauth2/client/OAuth2ClientProperties.Provider.html)Bean的Java文档。

由于我们已经将端口8080用于UAA，因此让我们在8081上运行它：

```properties
server.port=8081
```

### 4.2 登录

现在，如果我们访问/login路径，我们应该有一个所有已注册客户端的列表。在我们的例子中，我们只有一个已注册客户端：

![](/assets/images/2025/security/cloudfoundryuaa01.png)

**点击链接将会重定向到UAA登录页面**：

![](/assets/images/2025/security/cloudfoundryuaa02.png)

在这里，让我们使用appuser/appusersecret登录。

**提交表单后，我们将会重定向到批准表单，用户可以在其中授权或拒绝我们客户端的访问**：

![](/assets/images/2025/security/cloudfoundryuaa03.png)

然后，用户可以授予所需的权限。为了方便我们操作，**我们将选择除resource:write之外的所有内容**。

用户检查的任何内容都将是结果访问令牌中的范围。

为了证明这一点，我们可以复制index路径[http://localhost:8081](http://localhost:8081/)中显示的令牌，并使用[JWT调试器](https://jwt.io/#debugger-io)对其进行解码，**我们应该在批准页面上看到我们检查的范围**：

```json
{
    "jti": "f228d8d7486942089ff7b892c796d3ac",
    "sub": "0e6101d8-d14b-49c5-8c33-fc12d8d1cc7d",
    "scope": [
        "resource.read",
        "openid",
        "profile"
    ],
    "client_id": "webappclient"
    // more claims
}
```

一旦我们的客户端应用程序收到此令牌，它就可以对用户进行身份验证，并且用户将可以访问该应用程序。

现在，**一个不显示任何数据的应用程序并不是很有用，所以我们的下一步将是建立一个资源服务器**，它拥有用户的数据并将客户端连接到它。

完整的资源服务器将有两个受保护的API：一个需要resource.read范围，另一个需要resource.write。

我们将看到，**客户端使用我们授予的范围将能够调用read API，但不能调用write**。

## 5. 资源服务器

**资源服务器托管用户的受保护资源**。

它通过Authorization标头并与授权服务器(在我们的例子中是UAA)协商来验证客户端。

### 5.1 应用程序设置

为了创建资源服务器，我们将再次使用[Spring Initializr](https://start.spring.io/)来生成Spring Boot Web应用程序，这次，我们将选择Web和OAuth2资源服务器组件：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

与客户端应用程序一样，我们使用[Spring Boot 3.1.5](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-parent)版本。

**下一步是在application.properties文件中指示正在运行的CF UAA的位置**：

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/uaa/oauth/token
```

当然，我们在这里也选择一个新端口，8082就可以了：

```properties
server.port=8082
```

就这样，我们应该有一个可用的资源服务器，并且默认情况下，所有请求都需要在Authorization标头中提供有效的访问令牌。

### 5.2 保护资源服务器API

接下来，让我们添加一些值得保护的端点。

我们将添加一个具有两个端点的RestController，一个端点授权具有resource.read范围的用户，另一个端点授权具有resource.write范围的用户：

```java
@GetMapping("/read")
public String read(Principal principal) {
    return "Hello write: " + principal.getName();
}

@GetMapping("/write")
public String write(Principal principal) {
    return "Hello write: " + principal.getName();
}
```

接下来，我们将**覆盖默认的Spring Boot配置来保护这两个资源**：

```java
@EnableWebSecurity
public class CFUAAOAuth2ResourceServerSecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
                        .requestMatchers("/read/**")
                        .hasAuthority("SCOPE_resource.read")
                        .requestMatchers("/write/**")
                        .hasAuthority("SCOPE_resource.write")
                        .anyRequest()
                        .authenticated())
                .oauth2ResourceServer((oauth2) -> oauth2.jwt(Customizer.withDefaults()));
        return http.build();
    }
}
```

**请注意，当访问令牌中提供的范围转换为[Spring Security GrantedAuthority](https://www.baeldung.com/role-and-privilege-for-spring-security-registration)时，它们会以SCOPE_为前缀**。

### 5.3 从客户端请求受保护的资源

从客户端应用程序中，我们将使用RestTemplate调用两个受保护的资源，在发出请求之前，我们从上下文中检索访问令牌并将其添加到Authorization标头中：

```java
private String callResourceServer(OAuth2AuthenticationToken authenticationToken, String url) {
    OAuth2AuthorizedClient oAuth2AuthorizedClient = this.authorizedClientService.
            loadAuthorizedClient(authenticationToken.getAuthorizedClientRegistrationId(),
                    authenticationToken.getName());
    OAuth2AccessToken oAuth2AccessToken = oAuth2AuthorizedClient.getAccessToken();

    HttpHeaders headers = new HttpHeaders();
    headers.add("Authorization", "Bearer " + oAuth2AccessToken.getTokenValue());

    // call resource endpoint

    return response;
}
```

但请注意，如果我们使用[WebClient](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#servlet-webclient)而不是RestTemplate，我们可以删除这个样板。

然后，我们将向资源服务器端点添加两个调用：

```java
@GetMapping("/read")
public String read(OAuth2AuthenticationToken authenticationToken) {
    String url = remoteResourceServer + "/read";
    return callResourceServer(authenticationToken, url);
}

@GetMapping("/write")
public String write(OAuth2AuthenticationToken authenticationToken) {
    String url = remoteResourceServer + "/write";
    return callResourceServer(authenticationToken, url);
}
```

正如预期的那样，**/read API的调用将成功，但/write API的调用则不会成功**，HTTP状态403告诉我们用户未获得授权。

## 6. 总结

在本文中，我们首先简要概述了OAuth 2.0，因为它是OAuth 2.0授权服务器UAA的基础。然后，我们对其进行了配置，以便为客户端颁发访问令牌并保护资源服务器应用程序。