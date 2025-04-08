---
layout: post
title:  将JWT与Spring Security OAuth结合使用(旧堆栈)
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将讨论如何让我们的Spring Security OAuth2实现使用JSON Web Tokens。

我们还将继续在此OAuth系列[上一篇文章](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular)的基础上进行构建。

在我们开始之前，请注意以下重要事项。请记住，**Spring Security核心团队正在实现新的OAuth2堆栈，其中一些方面已经完成，而另一些方面仍在进行中**。

对于使用新Spring Security 5堆栈的本文版本，请查看我们的文章[使用JWT和Spring Security OAuth](https://www.baeldung.com/spring-security-oauth-jwt)。

## 2. Maven配置

首先，我们需要在pom.xml中添加spring-security-jwt依赖：

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
</dependency>
```

请注意，我们需要向授权服务器和资源服务器添加spring-security-jwt依赖。

## 3. 授权服务器

接下来，我们将配置授权服务器以使用JwtTokenStore-如下所示：

```java
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore())
                .accessTokenConverter(accessTokenConverter())
                .authenticationManager(authenticationManager);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("123");
        return converter;
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        return defaultTokenServices;
    }
}
```

请注意，我们在JwtAccessTokenConverter中使用对称密钥来签署我们的令牌，这意味着我们也需要对资源服务器使用完全相同的密钥。

## 4. 资源服务器

现在，让我们看一下资源服务器配置，它与授权服务器的配置非常相似：

```java
@Configuration
@EnableResourceServer
public class OAuth2ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(ResourceServerSecurityConfigurer config) {
        config.tokenServices(tokenServices());
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("123");
        return converter;
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        return defaultTokenServices;
    }
}
```

请记住，我们将这两个服务器定义为完全独立且可独立部署，这就是我们需要在新配置中再次声明一些相同Bean的原因。

## 5. 令牌中的自定义声明

现在让我们设置一些基础结构，以便能够在访问令牌中添加一些自定义声明。框架提供的标准声明都很好，但大多数时候我们需要令牌中的一些额外信息以便在客户端使用。

我们将定义一个TokenEnhancer来使用这些附加声明来定制我们的访问令牌。

在以下示例中，我们将使用CustomTokenEnhancer向我们的访问令牌添加一个额外的字段“organization”：

```java
public class CustomTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
        Map<String, Object> additionalInfo = new HashMap<>();
        additionalInfo.put(
                "organization", authentication.getName() + randomAlphabetic(4));
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(
                additionalInfo);
        return accessToken;
    }
}
```

然后，我们将其注入到授权服务器配置，如下所示：

```java
@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
    tokenEnhancerChain.setTokenEnhancers(
            Arrays.asList(tokenEnhancer(), accessTokenConverter()));

    endpoints.tokenStore(tokenStore())
            .tokenEnhancer(tokenEnhancerChain)
            .authenticationManager(authenticationManager);
}

@Bean
public TokenEnhancer tokenEnhancer() {
    return new CustomTokenEnhancer();
}
```

在这个新配置启动并运行后，令牌有效负载将如下所示：

```json
{
    "user_name": "john",
    "scope": [
        "foo",
        "read",
        "write"
    ],
    "organization": "johnIiCh",
    "exp": 1458126622,
    "authorities": [
        "ROLE_USER"
    ],
    "jti": "e0ad1ef3-a8a5-4eef-998d-00b26bc2c53f",
    "client_id": "fooClientIdPassword"
}
```

### 5.1 在JS客户端中使用访问令牌

最后，我们希望在AngualrJS客户端应用程序中使用令牌信息，为此，我们将使用[angular-jwt](https://github.com/auth0/angular-jwt)库。

因此，我们要做的就是在index.html中使用“organization”声明：

```html
<p class="navbar-text navbar-right">{{organization}}</p>

<script type="text/javascript"
        src="https://cdn.rawgit.com/auth0/angular-jwt/master/dist/angular-jwt.js">
</script>

<script>
    var app =
            angular.module('myApp', ["ngResource","ngRoute", "ngCookies", "angular-jwt"]);

    app.controller('mainCtrl', function($scope, $cookies, jwtHelper,...) {
        $scope.organiztion = "";

        function getOrganization(){
            var token = $cookies.get("access_token");
            var payload = jwtHelper.decodeToken(token);
            $scope.organization = payload.organization;
        }
    ...
    });
```

## 6. 访问资源服务器上的额外声明

但是，我们如何在资源服务器端访问这些信息？

我们在这里要做的是-从访问令牌中提取额外的声明：

```java
public Map<String, Object> getExtraInfo(OAuth2Authentication auth) {
    OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) auth.getDetails();
    OAuth2AccessToken accessToken = tokenStore
        .readAccessToken(details.getTokenValue());
    return accessToken.getAdditionalInformation();
}
```

在下一节中，我们将讨论如何使用自定义AccessTokenConverter将额外信息添加到我们的Authentication详细信息中。

### 6.1 自定义AccessTokenConverter

让我们创建CustomAccessTokenConverter并使用访问令牌声明设置身份验证详细信息：

```java
@Component
public class CustomAccessTokenConverter extends DefaultAccessTokenConverter {

    @Override
    public OAuth2Authentication extractAuthentication(Map<String, ?> claims) {
        OAuth2Authentication authentication =
                super.extractAuthentication(claims);
        authentication.setDetails(claims);
        return authentication;
    }
}
```

注意：DefaultAccessTokenConverter用于将身份验证详细信息设置为Null。

### 6.2 配置JwtTokenStore

接下来，我们将配置JwtTokenStore以使用我们的CustomAccessTokenConverter：

```java
@Configuration
@EnableResourceServer
public class OAuth2ResourceServerConfigJwt extends ResourceServerConfigurerAdapter {

    @Autowired
    private CustomAccessTokenConverter customAccessTokenConverter;

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setAccessTokenConverter(customAccessTokenConverter);
    }
    // ...
}
```

### 6.3 身份验证对象中可用的额外声明

现在授权服务器在令牌中添加了一些额外的声明，我们可以在资源服务器端直接在身份验证对象中进行访问：

```java
public Map<String, Object> getExtraInfo(Authentication auth) {
    OAuth2AuthenticationDetails oauthDetails = (OAuth2AuthenticationDetails) auth.getDetails();
    return (Map<String, Object>) oauthDetails
        .getDecodedDetails();
}
```

### 6.4 身份验证详情测试

让我们确保我们的身份验证对象包含额外的信息：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
        classes = ResourceServerApplication.class,
        webEnvironment = WebEnvironment.RANDOM_PORT)
public class AuthenticationClaimsIntegrationTest {

    @Autowired
    private JwtTokenStore tokenStore;

    @Test
    public void whenTokenDoesNotContainIssuer_thenSuccess() {
        String tokenValue = obtainAccessToken("fooClientIdPassword", "john", "123");
        OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);
        Map<String, Object> details = (Map<String, Object>) auth.getDetails();

        assertTrue(details.containsKey("organization"));
    }

    private String obtainAccessToken(
            String clientId, String username, String password) {

        Map<String, String> params = new HashMap<>();
        params.put("grant_type", "password");
        params.put("client_id", clientId);
        params.put("username", username);
        params.put("password", password);
        Response response = RestAssured.given()
                .auth().preemptive().basic(clientId, "secret")
                .and().with().params(params).when()
                .post("http://localhost:8081/spring-security-oauth-server/oauth/token");
        return response.jsonPath().getString("access_token");
    }
}
```

注意：我们从授权服务器获取了带有额外声明的访问令牌，然后从中读取Authentication对象，该详细信息对象中包含额外信息“organization”。

## 7. 非对称密钥对

在我们之前的配置中，我们使用对称密钥来签署我们的令牌：

```java
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("123");
    return converter;
}
```

我们还可以使用非对称密钥(公钥和私钥)进行签名过程。

### 7.1 生成JKS Java KeyStore文件

让我们首先使用命令行工具keytool生成密钥-更具体地说是.jks文件：

```shell
keytool -genkeypair -alias mytest 
                    -keyalg RSA 
                    -keypass mypass 
                    -keystore mytest.jks 
                    -storepass mypass
```

该命令将生成一个名为mytest.jks的文件，其中包含我们的密钥-公钥和私钥。

还要确保keypass和storepass相同。

### 7.2 导出公钥

接下来，我们需要从生成的JKS中导出我们的公钥，我们可以使用以下命令来执行此操作：

```shell
keytool -list -rfc --keystore mytest.jks | openssl x509 -inform pem -pubkey
```

示例响应如下：

```text
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAgIK2Wt4x2EtDl41C7vfp
OsMquZMyOyteO2RsVeMLF/hXIeYvicKr0SQzVkodHEBCMiGXQDz5prijTq3RHPy2
/5WJBCYq7yHgTLvspMy6sivXN7NdYE7I5pXo/KHk4nz+Fa6P3L8+L90E/3qwf6j3
DKWnAgJFRY8AbSYXt1d5ELiIG1/gEqzC0fZmNhhfrBtxwWXrlpUDT0Kfvf0QVmPR
xxCLXT+tEe1seWGEqeOLL5vXRLqmzZcBe1RZ9kQQm43+a9Qn5icSRnDfTAesQ3Cr
lAWJKl2kcWU1HwJqw+dZRSZ1X4kEXNMyzPdPBbGmU6MHdhpywI7SKZT7mX4BDnUK
eQIDAQAB
-----END PUBLIC KEY-----
-----BEGIN CERTIFICATE-----
MIIDCzCCAfOgAwIBAgIEGtZIUzANBgkqhkiG9w0BAQsFADA2MQswCQYDVQQGEwJ1
czELMAkGA1UECBMCY2ExCzAJBgNVBAcTAmxhMQ0wCwYDVQQDEwR0ZXN0MB4XDTE2
MDMxNTA4MTAzMFoXDTE2MDYxMzA4MTAzMFowNjELMAkGA1UEBhMCdXMxCzAJBgNV
BAgTAmNhMQswCQYDVQQHEwJsYTENMAsGA1UEAxMEdGVzdDCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAICCtlreMdhLQ5eNQu736TrDKrmTMjsrXjtkbFXj
Cxf4VyHmL4nCq9EkM1ZKHRxAQjIhl0A8+aa4o06t0Rz8tv+ViQQmKu8h4Ey77KTM
urIr1zezXWBOyOaV6Pyh5OJ8/hWuj9y/Pi/dBP96sH+o9wylpwICRUWPAG0mF7dX
eRC4iBtf4BKswtH2ZjYYX6wbccFl65aVA09Cn739EFZj0ccQi10/rRHtbHlhhKnj
iy+b10S6ps2XAXtUWfZEEJuN/mvUJ+YnEkZw30wHrENwq5QFiSpdpHFlNR8CasPn
WUUmdV+JBFzTMsz3TwWxplOjB3YacsCO0imU+5l+AQ51CnkCAwEAAaMhMB8wHQYD
VR0OBBYEFOGefUBGquEX9Ujak34PyRskHk+WMA0GCSqGSIb3DQEBCwUAA4IBAQB3
1eLfNeq45yO1cXNl0C1IQLknP2WXg89AHEbKkUOA1ZKTOizNYJIHW5MYJU/zScu0
yBobhTDe5hDTsATMa9sN5CPOaLJwzpWV/ZC6WyhAWTfljzZC6d2rL3QYrSIRxmsp
/J1Vq9WkesQdShnEGy7GgRgJn4A8CKecHSzqyzXulQ7Zah6GoEUD+vjb+BheP4aN
hiYY1OuXD+HsdKeQqS+7eM5U7WW6dz2Q8mtFJ5qAxjY75T0pPrHwZMlJUhUZ+Q2V
FfweJEaoNB9w9McPe1cAiE+oeejZ0jq0el3/dJsx3rlVqZN+lMhRJJeVHFyeb3XF
lLFCUGhA7hxn2xf3x1JW
-----END CERTIFICATE-----
```

我们只获取公钥并将其复制到我们的资源服务器src/main/resources/public.txt：

```text
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAgIK2Wt4x2EtDl41C7vfp
OsMquZMyOyteO2RsVeMLF/hXIeYvicKr0SQzVkodHEBCMiGXQDz5prijTq3RHPy2
/5WJBCYq7yHgTLvspMy6sivXN7NdYE7I5pXo/KHk4nz+Fa6P3L8+L90E/3qwf6j3
DKWnAgJFRY8AbSYXt1d5ELiIG1/gEqzC0fZmNhhfrBtxwWXrlpUDT0Kfvf0QVmPR
xxCLXT+tEe1seWGEqeOLL5vXRLqmzZcBe1RZ9kQQm43+a9Qn5icSRnDfTAesQ3Cr
lAWJKl2kcWU1HwJqw+dZRSZ1X4kEXNMyzPdPBbGmU6MHdhpywI7SKZT7mX4BDnUK
eQIDAQAB
-----END PUBLIC KEY-----
```

或者，我们可以通过添加-noout参数仅导出公钥：

```shell
keytool -list -rfc --keystore mytest.jks | openssl x509 -inform pem -pubkey -noout
```

### 7.3 Maven配置

接下来，我们不希望JKS文件被Maven过滤过程选中-因此我们要确保在pom.xml中将其排除：

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <excludes>
                <exclude>*.jks</exclude>
            </excludes>
        </resource>
    </resources>
</build>
```

如果我们使用Spring Boot，我们需要确保JKS文件通过Spring Boot Maven插件–addResources添加到应用程序类路径中：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 7.4 授权服务器

现在，我们将配置JwtAccessTokenConverter以使用来自mytest.jks的密钥对，如下所示：

```java
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    KeyStoreKeyFactory keyStoreKeyFactory = new KeyStoreKeyFactory(new ClassPathResource("mytest.jks"), "mypass".toCharArray());
    converter.setKeyPair(keyStoreKeyFactory.getKeyPair("mytest"));
    return converter;
}
```

### 7.5 资源服务器

最后，我们需要配置我们的资源服务器以使用公钥，如下所示：

```java
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    Resource resource = new ClassPathResource("public.txt");
    String publicKey = null;
    try {
        publicKey = IOUtils.toString(resource.getInputStream());
    } catch (final IOException e) {
        throw new RuntimeException(e);
    }
    converter.setVerifierKey(publicKey);
    return converter;
}
```

## 8. 总结

在这篇简短的文章中，我们重点介绍了如何设置Spring Security OAuth2项目以使用JSON Web Tokens。