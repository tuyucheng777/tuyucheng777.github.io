---
layout: post
title:  将JWT与Spring Security OAuth结合使用
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将讨论如何让我们的Spring Security OAuth2实现使用JSON Web Tokens。

我们还将继续在此OAuth系列中基于[Spring REST API + OAuth2 + Angular](https://www.baeldung.com/rest-api-spring-oauth2-angular)文章构建。

## 2. OAuth2授权服务器

以前，Spring Security OAuth堆栈提供了将授权服务器设置为Spring应用程序的可能性，然后我们必须将其配置为使用JwtTokenStore，以便我们可以使用JWT令牌。

但是，OAuth堆栈已被Spring弃用，现在我们将使用Keycloak作为我们的授权服务器。

**因此，这次我们将在Spring Boot应用程序中将授权服务器设置为[嵌入式Keycloak服务器](https://www.baeldung.com/keycloak-embedded-in-a-spring-boot-application)**。它默认颁发JWT令牌，因此不需要在这方面进行任何其他配置。

## 3. 资源服务器

现在让我们看看如何配置我们的资源服务器以使用JWT。

我们将在application.yml文件中执行此操作：

```yaml
server:
    port: 8081
    servlet:
        context-path: /resource-server

spring:
    jpa:
        defer-datasource-initialization: true
    security:
        oauth2:
            resourceserver:
                jwt:
                    issuer-uri: http://localhost:8083/auth/realms/tuyucheng
                    jwk-set-uri: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/certs
```

JWT包含Token内的所有信息，因此资源服务器需要验证Token的签名以确保数据未被修改，**jwk-set-uri属性包含服务器可用于此目的的公钥**。

issuer-uri属性指向基本授权服务器URI，它还可用于验证iss声明作为附加的安全措施。

此外，如果未设置jwk-set-uri属性，资源服务器将尝试使用issuer-uri从[授权服务器元数据端点](https://tools.ietf.org/html/rfc8414#section-3)确定此密钥的位置。

值得注意的是，**添加issuer-uri属性要求我们在启动资源服务器应用程序之前必须运行授权服务器**。

现在让我们看看如何使用Java配置来配置JWT支持： 

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors()
                .and()
                .authorizeRequests()
                .antMatchers(HttpMethod.GET, "/user/info", "/api/foos/**")
                .hasAuthority("SCOPE_read")
                .antMatchers(HttpMethod.POST, "/api/foos")
                .hasAuthority("SCOPE_write")
                .anyRequest()
                .authenticated()
                .and()
                .oauth2ResourceServer()
                .jwt();
    }
}
```

在这里，我们覆盖了默认的Http安全配置；**我们需要明确指定我们希望它作为资源服务器运行，并且我们将分别使用方法oauth2ResourceServer()和jwt()使用JWT格式的访问令牌**。

上述JWT配置是默认的Spring Boot实例提供给我们的，我们稍后会看到，它也可以自定义。

## 4. 令牌中的自定义声明

现在让我们设置一些基础结构，以便能够在授权服务器返回的访问令牌中添加一些自定义声明。框架提供的标准声明都很好，但大多数时候我们需要在令牌中添加一些额外信息以便在客户端使用。

让我们举一个自定义声明的例子-organization，它将包含给定用户的组织的名称。

### 4.1 授权服务器配置

为此，我们需要在声明定义文件tuyucheng-realm.json中添加几个配置：

- 为我们的用户john@test.com添加一个属性organization：

  ```json
  "attributes": {
      "organization" : "tuyucheng"
  },
  ```

- 向jwtClient配置中添加一个名为organization的protocolMapper：

  ```json
  "protocolMappers": [{
      "id": "06e5fc8f-3553-4c75-aef4-5a4d7bb6c0d1",
      "name": "organization",
      "protocol": "openid-connect",
      "protocolMapper": "oidc-usermodel-attribute-mapper",
      "consentRequired": false,
      "config": {
          "userinfo.token.claim": "true",
          "user.attribute": "organization",
          "id.token.claim": "true",
          "access.token.claim": "true",
          "claim.name": "organization",
          "jsonType.label": "String"
      }
  }],
  ```

对于独立的Keycloak设置，也可以使用管理控制台完成此操作。

重要的是要记住，**上述JSON配置特定于Keycloak，并且可能因其他OAuth服务器而异**。

启动并运行此新配置后，我们将在john@test.com的令牌有效负载中获得一个额外的属性organization = tuyucheng：

```json
{
    jti: "989ce5b7-50b9-4cc6-bc71-8f04a639461e"
    exp: 1585242462
    nbf: 0
    iat: 1585242162
    iss: "http://localhost:8083/auth/realms/tuyucheng"
    sub: "a5461470-33eb-4b2d-82d4-b0484e96ad7f"
    typ: "Bearer"
    azp: "jwtClient"
    auth_time: 1585242162
    session_state: "384ca5cc-8342-429a-879c-c15329820006"
    acr: "1"
    scope: "profile write read"
    organization: "tuyucheng"
    preferred_username: "john@test.com"
}
```

### 4.2 在Angular客户端中使用访问令牌

接下来，我们将要在Angular客户端应用程序中使用Token信息。为此，我们将使用[angular2-jwt](https://github.com/auth0/angular2-jwt)库。

我们将在AppService中使用organization声明，并添加函数getOrganization：

```typescript
getOrganization() {
    var token = Cookie.get("access_token");
    var payload = this.jwtHelper.decodeToken(token);
    this.organization = payload.organization; 
    return this.organization;
}
```

此函数利用angular2-jwt库中的JwtHelperService来解码访问令牌并获取我们的自定义声明，现在我们需要做的就是在我们的AppComponent中显示它：

```typescript
@Component({
    selector: 'app-root',
    template: `<nav class="navbar navbar-default">
        <div class="container-fluid">
            <div class="navbar-header">
                <a class="navbar-brand" href="/">Spring Security Oauth - Authorization Code</a>
            </div>
        </div>
        <div class="navbar-brand">
            <p>{{organization}}</p>
        </div>
</nav>
<router-outlet></router-outlet>`
})

export class AppComponent implements OnInit {
    public organization = "";
    constructor(private service: AppService) { }
    
    ngOnInit() {
        this.organization = this.service.getOrganization();
    }
}
```

## 5. 访问资源服务器中的额外声明

但是我们如何在资源服务器端访问这些信息呢？

### 5.1 访问认证服务器声明

这很简单，我们只需要**从org.springframework.security.oauth2.jwt.Jwt的AuthenticationPrincipal中提取它**，就像我们对UserInfoController中的任何其他属性所做的那样：

```java
@GetMapping("/user/info")
public Map<String, Object> getUserInfo(@AuthenticationPrincipal Jwt principal) {
    Map<String, String> map = new Hashtable<String, String>();
    map.put("user_name", principal.getClaimAsString("preferred_username"));
    map.put("organization", principal.getClaimAsString("organization"));
    return Collections.unmodifiableMap(map);
}
```

### 5.2 添加/删除/重命名声明的配置

现在如果我们想在资源服务器端添加更多声明，或者删除或重命名一些声明该怎么办？

假设我们想要修改来自身份验证服务器的organization声明，使其值为大写。但是，如果用户没有该声明，我们需要将其值设置为unknown。

为了实现这一点，我们必须**添加一个实现Converter接口并使用MappedJwtClaimSetConverter来转换声明的类**：

```java
public class OrganizationSubClaimAdapter implements Converter<Map<String, Object>, Map<String, Object>> {
    
    private final MappedJwtClaimSetConverter delegate = MappedJwtClaimSetConverter.withDefaults(Collections.emptyMap());

    public Map<String, Object> convert(Map<String, Object> claims) {
        Map<String, Object> convertedClaims = this.delegate.convert(claims);
        String organization = convertedClaims.get("organization") != null ? (String) convertedClaims.get("organization") : "unknown";
        
        convertedClaims.put("organization", organization.toUpperCase());
        return convertedClaims;
    }
}
```

然后，在我们的SecurityConfig类中，我们需要添加我们自己的JwtDecoder实例来覆盖Spring Boot提供的实例，并**将我们的OrganizationSubClaimAdapter设置为其声明转换器**：

```java
@Bean
public JwtDecoder jwtDecoder(OAuth2ResourceServerProperties properties) {
    NimbusJwtDecoder jwtDecoder = NimbusJwtDecoder.withJwkSetUri(properties.getJwt().getJwkSetUri()).build();
    
    jwtDecoder.setClaimSetConverter(new OrganizationSubClaimAdapter());
    return jwtDecoder;
}
```

现在，当我们对用户mike@other.com访问/user/infoAPI时，我们将得到organization信息为UNKNOWN。

请注意，覆盖Spring Boot配置的默认JwtDecoder Bean时应小心谨慎，以确保仍然包含所有必要的配置。

## 6. 从Java Keystore加载密钥

在我们之前的配置中，我们使用授权服务器的默认公钥来验证我们的令牌的完整性。

我们还可以使用存储在Java Keystore文件中的密钥对和证书来进行签名过程。

### 6.1 生成JKS Java KeyStore文件

让我们首先使用命令行工具keytool生成密钥，更具体地说是.jks文件：

```shell
keytool -genkeypair -alias mytest 
                    -keyalg RSA 
                    -keypass mypass 
                    -keystore mytest.jks 
                    -storepass mypass
```

该命令将生成一个名为mytest.jks的文件，其中包含我们的密钥、公钥和私钥。

还要确保keypass和storepass相同。

### 6.2 导出公钥

接下来，我们需要从生成的JKS中导出公钥，我们可以使用以下命令来执行此操作：

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

### 6.3 Maven配置

我们不希望JKS文件被Maven过滤过程选中，因此我们要确保在pom.xml中将其排除：

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

如果我们使用Spring Boot，我们需要确保我们的JKS文件通过Spring Boot Maven插件addResources添加到应用程序类路径中：

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

### 6.4 授权服务器

现在我们将配置Keycloak以使用来自mytest.jks的密钥对，方法是将其添加到声明定义JSON文件的KeyProvider部分，如下所示：

```json
{
    "id": "59412b8d-aad8-4ab8-84ec-e546900fc124",
    "name": "java-keystore",
    "providerId": "java-keystore",
    "subComponents": {},
    "config": {
        "keystorePassword": [ "mypass" ],
        "keyAlias": [ "mytest" ],
        "keyPassword": [ "mypass" ],
        "active": [ "true" ],
        "keystore": [
                "src/main/resources/mytest.jks"
              ],
        "priority": [ "101" ],
        "enabled": [ "true" ],
        "algorithm": [ "RS256" ]
    }
},
```

在这里，我们将priority设置为101，高于我们授权服务器的任何其他密钥对，并将active设置为true，这样做是为了确保我们的资源服务器将从我们之前指定的jwk-set-uri属性中选择这个特定的密钥对。

再次强调，此配置特定于Keycloak，并且可能与其他OAuth服务器实现有所不同。

## 7. 总结

在这篇简短的文章中，我们重点介绍了如何设置Spring Security OAuth2项目以使用JSON Web Tokens。