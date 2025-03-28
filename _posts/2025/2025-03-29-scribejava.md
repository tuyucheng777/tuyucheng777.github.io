---
layout: post
title:  ScribeJava指南
category: libraries
copyright: libraries
excerpt: ScribeJava
---

## 1. 简介

在本教程中，我们将研究[ScribeJava](https://github.com/scribejava/scribejava)库。

**ScribeJava是一个简单的Java OAuth客户端，可帮助管理OAuth流程**。

该库的主要特点是它支持所有主要的1.0和2.0 OAuth API。此外，如果我们必须使用不受支持的API，该库提供了几个类来实现我们的OAuth API。

另一个重要功能是可以选择使用哪个客户端。事实上，ScribeJava支持多个HTTP客户端：

- [Async Http Client](https://www.baeldung.com/async-http-client)
- [OkHttp](https://www.baeldung.com/guide-to-okhttp)
- [Apache HttpComponents HttpClient](https://hc.apache.org/httpcomponents-client-5.1.x/)

此外，该库是线程安全的并且与Java 7兼容，因此我们可以在旧环境中使用它。

## 2. 依赖

**ScribeJava被组织成核心和API模块**，后者包括一组外部API(Google、GitHub、Twitter等)和核心工件：

```xml
<dependency>
    <groupId>com.github.scribejava</groupId>
    <artifactId>scribejava-apis</artifactId>
    <version>latest-version</version>
</dependency>
```

如果我们只需要核心类而不需要任何外部API，那么我们只需要提取核心模块：

```xml
<dependency>
    <groupId>com.github.scribejava</groupId>
    <artifactId>scribejava-core</artifactId>
    <version>latest-version</version>
</dependency>
```

最新版本可以在[Maven仓库](https://mvnrepository.com/search?q=scribejava-core)中找到。

## 3. OAuthService

**该库的主要部分是抽象类OAuthService**，它包含正确管理OAuth的“握手”所需的所有参数。

根据协议的版本，我们将分别对[OAuth 1.0](https://tools.ietf.org/html/rfc5849)和[OAuth 2.0](https://tools.ietf.org/html/rfc6749)使用Oauth10Service或Oauth20Service具体类。

为了构建OAuthService实现，该库提供了一个ServiceBuilder：

```java
OAuthService service = new ServiceBuilder("api_key")
    .apiSecret("api_secret")
    .scope("scope")
    .callback("callback")
    .build(GoogleApi20.instance());
```

我们应该设置授权服务器提供的api_key和api_secret令牌。

此外，我们可以设置请求的范围和授权服务器在授权流程结束时应将用户重定向到的回调。

请注意，根据协议版本的不同，并非所有参数都是强制的。

最后，我们必须调用build()方法来构建OAuthService，并向其传递我们要使用的API实例。我们可以在ScribeJava [GitHub](https://github.com/scribejava/scribejava)上找到所支持的API的完整列表。

### 3.1 HTTP客户端

此外，**该库允许我们选择使用哪个HTTP客户端**：

```java
ServiceBuilder builder = new ServiceBuilder("api_key")
    .httpClient(new OkHttpHttpClient());
```

当然，之后我们已经包含了前面示例所需的依赖：

```xml
<dependency>
    <groupId>com.github.scribejava</groupId>
    <artifactId>scribejava-httpclient-okhttp</artifactId>
    <version>latest-version</version>
</dependency>
```

最新版本可以在[Maven仓库](https://mvnrepository.com/artifact/com.github.scribejava)中找到。

### 3.2 调试模式

此外，**我们还可以使用调试模式来帮助我们排除故障**：

```java
ServiceBuilder builder = new ServiceBuilder("api_key")
    .debug();
```

我们只需调用debug()方法，Debug会将一些相关信息输出到System.out。

另外，如果我们想使用不同的输出，还有另一种方法可以接收OutputStream来发送调试信息：

```java
FileOutputStream debugFile = new FileOutputStream("debug");

ServiceBuilder builder = new ServiceBuilder("api_key")
    .debug()
    .debugStream(debugFile);
```

## 4. OAuth 1.0流程

现在让我们关注如何处理OAuth1流。

在这个例子中，**我们将使用Twitter API获取access token，并使用它发出请求**。

首先，我们必须使用构建起构建Oauth10Service，就像我们之前看到的一样：

```java
OAuth10aService service = new ServiceBuilder("api_key")
    .apiSecret("api_secret")
    .build(TwitterApi.instance());
```

一旦我们有了OAuth10Service，我们就可以获取一个requestToken并使用它来获取授权URL：

```java
OAuth1RequestToken requestToken = service.getRequestToken();
String authUrl = service.getAuthorizationUrl(requestToken);
```

此时需要将用户重定向到authUrl，并获取页面提供的oauthVerifier。

因此，我们使用oauthVerifier来获取accessToken：

```java
OAuth1AccessToken accessToken = service.getAccessToken(requestToken,oauthVerifier);
```

最后，我们可以使用OAuthRequest对象创建一个请求，并使用signRequest()方法将令牌添加到其中：

```java
OAuthRequest request = new OAuthRequest(Verb.GET, 
    "https://api.twitter.com/1.1/account/verify_credentials.json");
service.signRequest(accessToken, request);

Response response = service.execute(request);
```

执行该 请求后，我们得到一个Response对象。

## 5. OAuth 2.0流程

OAuth 2.0流程与OAuth 1.0没有太大区别，为了解释这些变化，**我们将使用Google API获取access token**。

与OAuth 1.0流程中一样，我们必须构建OAuthService并获取authUrl，但这次我们将使用OAuth20Service实例：

```java
OAuth20Service service = new ServiceBuilder("api_key")
    .apiSecret("api_secret")
    .scope("https://www.googleapis.com/auth/userinfo.email")
    .callback("http://localhost:8080/auth")
    .build(GoogleApi20.instance());

String authUrl = service.getAuthorizationUrl();
```

请注意，在这种情况下，我们需要提供请求的scope和在授权流程结束时联系我们的回调。

同样，我们必须将用户重定向到authUrl，并在回调的url中获取code参数：

```java
OAuth2AccessToken accessToken = service.getAccessToken(code);

OAuthRequest request = new OAuthRequest(Verb.GET, "https://www.googleapis.com/oauth2/v1/userinfo?alt=json");
service.signRequest(accessToken, request);

Response response = service.execute(request);
```

最后，为了发出请求，我们使用getAccessToken()方法获取accessToken。

## 6. 自定义API

我们可能需要使用ScribeJava不支持的API。在这种情况下，**该库允许我们实现自己的API**。

我们唯一需要做的就是提供DefaultApi10或DefaultApi20类的实现。

假设我们有一个带密码授予的OAuth 2.0授权服务器，在这种情况下，我们可以实现DefaultApi20以便获取访问令牌：

```java
public class MyApi extends DefaultApi20 {

    public MyApi() {}

    private static class InstanceHolder {
        private static final MyApi INSTANCE = new MyApi();
    }

    public static MyApi instance() {
        return InstanceHolder.INSTANCE;
    }

    @Override
    public String getAccessTokenEndpoint() {
        return "http://localhost:8080/oauth/token";
    }

    @Override
    protected String getAuthorizationBaseUrl() {
        return null;
    }
}
```

因此，我们可以按照与之前类似的方式获取访问令牌：

```java
OAuth20Service service = new ServiceBuilder("tuyucheng_api_key")
    .apiSecret("tuyucheng_api_secret")
    .scope("read write")
    .build(MyApi.instance());

OAuth2AccessToken token = service.getAccessTokenPasswordGrant(username, password);

OAuthRequest request = new OAuthRequest(Verb.GET, "http://localhost:8080/me");
service.signRequest(token, request);
Response response = service.execute(request);
```

## 7. 总结

在本文中，我们了解了ScribeJava提供的现成的最有用的类。

我们学习了如何使用外部API处理OAuth 1.0和OAuth 2.0流程，我们还学习了如何配置库以使用我们自己的API。