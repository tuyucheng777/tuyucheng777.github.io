---
layout: post
title:  使用Auth0管理JWT java-jwt
category: security
copyright: security
excerpt: java-jwt
---

## 1. 简介

[JWT](https://www.rfc-editor.org/rfc/rfc7519)(JSON Web Token)是一种标准，它定义了一种紧凑且安全的方式，用于在双方之间传输数据和签名。JWT中的有效负载是一个JSON对象，它声明了一些声明。由于此有效负载是经过数字签名的，因此验证者可以轻松验证和信任它，**JWT可以使用密钥或公钥/私钥对进行签名**。

在本教程中，我们将学习如何使用[Auth0 JWT Java库](https://github.com/auth0/java-jwt)[创建和解码JWT](https://www.baeldung.com/java-jwt-token-decode)。

## 2. JWT的结构

JWT基本上由三部分组成：

- 标头
- 有效载荷
- 签名

每个部分代表一个[Base64编码](https://www.baeldung.com/java-base64-encode-and-decode)的字符串，并以点(‘.’)作为分隔符。

### 2.1 JWT标头

JWT标头通常由两部分组成：令牌类型(即“JWT”)和用于签署JWT的签名算法。

Auth0 Java JWT库提供了各种算法实现来签署JWT，如HMAC、RSA和ECDSA。

我们来看一个示例JWT标头：

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

然后对上述标头对象进行Base64编码，以形成JWT的第一部分。

### 2.2 JWT负载

JWT负载包含一组声明，声明基本上是关于实体的陈述以及一些附加数据。

声明类型有三种：

- registered：这是一组有用的预定义声明，建议使用，但不是强制性的。为了保持JWT的紧凑性，这些声明名称仅包含3个字符。一些已注册声明包括iss(颁发者)、exp(到期时间)和sub(主题)等。
- public：JWT使用者可以随意定义这些。
- private：我们可以使用这些声明来创建自定义声明。

我们来看一个JWT负载示例：

```json
{
    "sub": "Tuyucheng Details",
    "nbf": 1669463994,
    "iss": "Tuyucheng",
    "exp": 1669463998,
    "userId": "1234",
    "iat": 1669463993,
    "jti": "b44bd6c6-f128-4415-8458-6d8b4bc98e4a"
}
```

在这里，我们可以看到负载包含一个私有声明userId，表示登录用户的ID。此外，我们还可以找到一些其他有用的受限声明，它们定义了有关JWT的其他详细信息。

然后对JWT负载进行Base64编码，形成JWT的第二部分。

### 2.3 JWT签名

最后，**我们使用密钥对编码后的标头和有效负载进行签名，生成JWT签名**，然后可以使用签名来验证JWT中的数据是否有效。

需要注意的是，**任何有权访问JWT的人都可以轻松解码和查看其内容**。签名的令牌可以验证其中包含的声明的完整性，如果使用公钥/私钥对对令牌进行签名，则签名还会证明只有持有私钥的一方才是签名者。

最后，将这三个部分结合起来，我们就得到了JWT：

```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJCYWVsZHVuZyBEZXRhaWxzIiwibmJmIjoxNjY5NDYzOTk0LCJpc3MiOiJCYWVsZHVuZyIsImV4cCI6MTY2OTQ2Mzk5OCwidXNlcklkIjoiMTIzNCIsImlhdCI6MTY2OTQ2Mzk5MywianRpIjoiYjQ0YmQ2YzYtZjEyOC00NDE1LTg0NTgtNmQ4YjRiYzk4ZTRhIn0.14jm1FVPXFDJCUBARDTQkUErMmUTqdt5uMTGW6hDuV0
```

接下来，让我们看看如何使用Auth0 Java JWT库创建和管理JWT。

## 3. 使用Auth0

Auth0提供了一个易于使用的Java库来创建和管理JWT。

### 3.1 依赖

首先，我们将Auth0 Java JWT库的[Maven依赖](https://mvnrepository.com/artifact/com.auth0/java-jwt)添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.2.1</version>
</dependency>
```

### 3.2 配置算法和验证器

我们首先创建Algorithm类的实例，在本教程中，我们将使用HMAC256算法来签署我们的JWT：

```java
Algorithm algorithm = Algorithm.HMAC256("tuyucheng");
```

在这里，我们用密钥初始化Algorithm的一个实例，稍后我们将在创建和验证令牌时使用它。

此外，让我们初始化JWTVerifier的一个实例，我们将使用它来验证创建的令牌：

```java
JWTVerifier verifier = JWT.require(algorithm)
    .withIssuer("Tuyucheng")
    .build();
```

**要初始化验证器，我们使用JWT.require(Algorithm)方法**，此方法返回Verification的一个实例，然后我们可以使用它来构建JWTVerifier的实例。

我们现在准备创建我们的JWT。

### 3.3 创建JWT

**要创建JWT，我们使用JWT.create()方法**，该方法返回JWTCreator.Builder类的实例，我们将使用此Builder类通过使用Algorithm实例对声明进行签名来构建JWT令牌：

```java
String jwtToken = JWT.create()
    .withIssuer("Tuyucheng")
    .withSubject("Tuyucheng Details")
    .withClaim("userId", "1234")
    .withIssuedAt(new Date())
    .withExpiresAt(new Date(System.currentTimeMillis() + 5000L))
    .withJWTId(UUID.randomUUID()
        .toString())
    .withNotBefore(new Date(System.currentTimeMillis() + 1000L))
    .sign(algorithm);
```

上面的代码片段返回一个JWT：

```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJCYWVsZHVuZyBEZXRhaWxzIiwibmJmIjoxNjY5NDYzOTk0LCJpc3MiOiJCYWVsZHVuZyIsImV4cCI6MTY2OTQ2Mzk5OCwidXNlcklkIjoiMTIzNCIsImlhdCI6MTY2OTQ2Mzk5MywianRpIjoiYjQ0YmQ2YzYtZjEyOC00NDE1LTg0NTgtNmQ4YjRiYzk4ZTRhIn0.14jm1FVPXFDJCUBARDTQkUErMmUTqdt5uMTGW6hDuV0
```

让我们讨论一下上面用来设置一些声明的一些JWTCreator.Builder类方法：

- withIssuer()：识别创建并签署令牌的一方
- withSubject()：标识JWT的主题
- withIssuedAt()：标识JWT的创建时间；我们可以使用它来确定JWT的使用年限
- withExpiresAt()：标识JWT的过期时间
- withJWTId()：JWT的唯一标识符
- withNotBefore()：确定JWT不应被接受处理的时间
- withClaim()：用于设置任何自定义声明

### 3.4 验证JWT

此外，**为了验证JWT，我们使用之前初始化的JWTVerifier中的JWTVerifier.verify(String)方法**。如果JWT有效，该方法将解析JWT并返回DecodedJWT的实例。

DecodedJWT实例提供了各种便捷方法，我们可以使用它们来获取JWT中包含的声明。如果JWT无效，该方法将抛出JWTVerificationException。

让我们解码之前创建的JWT：

```java
try {
    DecodedJWT decodedJWT = verifier.verify(jwtToken);
} catch (JWTVerificationException e) {
    System.out.println(e.getMessage());
}
```

一旦我们获得DecodedJWT实例，我们就可以使用其各种Getter方法来获取声明。

例如，要获取自定义声明，我们使用DecodedJWT.getClaim(String)方法，此方法返回Claim的实例：

```java
Claim claim = decodedJWT.getClaim("userId");
```

在这里，我们获取之前创建JWT时设置的自定义声明userId。现在，我们可以通过调用Claim.asString()或任何其他基于声明数据类型的可用方法来获取声明值：

```java
String userId = claim.asString();
```

上面的代码片段返回我们自定义声明的字符串“1234” 。

除了Auth0 Java JWT库之外，Auth0还提供了直观的基于Web的[JWT调试器](https://jwt.io/)来帮助我们解码和验证JWT。

## 4. 总结

在本文中，我们研究了JWT的结构以及如何使用它进行身份验证。

然后，我们使用Auth0 Java JWT库来创建令牌，并使用其签名、算法和密钥来验证令牌的完整性。