---
layout: post
title:  Spring REST API OAuth2 - 在Angular中处理刷新令牌
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将继续探索在[上一篇文章](https://www.baeldung.com/rest-api-spring-oauth2-angular)中开始整理的OAuth2授权码流程，**并重点介绍如何在Angular应用中处理刷新令牌。我们还将使用Zuul代理**。

**我们将在Spring Security 5中使用OAuth堆栈**，如果你想使用Spring Security OAuth旧堆栈，请查看之前的文章：[Spring REST API OAuth2 – 在AngularJS中处理刷新令牌(旧OAuth堆栈)](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular-js-legacy)

## 2. 访问令牌过期

首先，请记住，客户端使用授权码授权类型分两步获取访问令牌。第一步，[我们获取授权码](https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service)。第二步，我们实际[获取访问令牌](https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service-1)。

我们的访问令牌存储在Cookie中，该令牌的过期时间取决于令牌本身的过期时间：

```javascript
var expireDate = new Date().getTime() + (1000 * token.expires_in);
Cookie.set("access_token", token.access_token, expireDate);
```

需要了解的是，Cookie本身仅用于存储，不会在OAuth2流程中驱动任何其他操作。例如，浏览器永远不会自动将Cookie与请求一起发送到服务器，因此我们在这里是安全的。

但请注意我们实际上是如何定义这个retrieveToken()函数来获取访问令牌的：

```typescript
retrieveToken(code) {
    let params = new URLSearchParams();
    params.append('grant_type','authorization_code');
    params.append('client_id', this.clientId);
    params.append('client_secret', 'newClientSecret');
    params.append('redirect_uri', this.redirectUri);
    params.append('code',code);

    let headers = new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});

    this._http.post('http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/token',
        params.toString(), { headers: headers })
        .subscribe(
            data => this.saveToken(data),
            err => alert('Invalid Credentials'));
}
```

我们在params中发送客户端密钥，这实际上不是一种安全的处理方式，让我们看看如何避免这样做。

## 3. 代理

因此，**我们现在将在前端应用程序中运行一个Zuul代理，它基本上位于前端客户端和授权服务器之间**，所有敏感信息都将在此层处理。

前端客户端现在将作为Boot应用程序托管，以便我们可以使用Spring Cloud Zuul Starter无缝连接到我们嵌入式Zuul代理。

如果你想了解Zuul的基础知识，请快速阅读[主要Zuul文章](https://www.baeldung.com/spring-rest-with-zuul-proxy)。

现在**让我们配置代理的路由**：

```yaml
zuul:
    routes:
        auth/code:
            path: /auth/code/**
            sensitiveHeaders:
            url: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/auth
        auth/token:
            path: /auth/token/**
            sensitiveHeaders:
            url: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/token
        auth/refresh:
            path: /auth/refresh/**
            sensitiveHeaders:
            url: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/token
        auth/redirect:
            path: /auth/redirect/**
            sensitiveHeaders:
            url: http://localhost:8089/
        auth/resources:
            path: /auth/resources/**
            sensitiveHeaders:
            url: http://localhost:8083/auth/resources/
```

我们已经设置了路由来处理以下情况：

- auth/code：获取授权码并将其保存在Cookie中
- auth/redirect：处理重定向到授权服务器的登录页面
- auth/resources：映射到授权服务器的登录页面资源(css和js)的相应路径
- auth/token：获取访问令牌，从有效负载中删除refresh_token并将其保存在Cookie中
- auth/refresh：获取刷新令牌，将其从有效负载中删除并将其保存在Cookie中

有趣的是，我们只代理授权服务器的流量，而不代理其他流量。我们真正需要的只是在客户端获取新令牌时才使用代理。

接下来我们来逐一看一下。

## 4. 使用Zuul PreFilter获取代码

**代理的第一次使用很简单-我们设置一个请求来获取授权码**：

```java
@Component
public class CustomPreZuulFilter extends ZuulFilter {
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest req = ctx.getRequest();
        String requestURI = req.getRequestURI();
        if (requestURI.contains("auth/code")) {
            Map<String, List> params = ctx.getRequestQueryParams();
            if (params == null) {
                params = Maps.newHashMap();
            }
            params.put("response_type", Lists.newArrayList(new String[] { "code" }));
            params.put("scope", Lists.newArrayList(new String[] { "read" }));
            params.put("client_id", Lists.newArrayList(new String[] { CLIENT_ID }));
            params.put("redirect_uri", Lists.newArrayList(new String[] { REDIRECT_URL }));
            ctx.setRequestQueryParams(params);
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        boolean shouldfilter = false;
        RequestContext ctx = RequestContext.getCurrentContext();
        String URI = ctx.getRequest().getRequestURI();

        if (URI.contains("auth/code") || URI.contains("auth/token") ||
                URI.contains("auth/refresh")) {
            shouldfilter = true;
        }
        return shouldfilter;
    }

    @Override
    public int filterOrder() {
        return 6;
    }

    @Override
    public String filterType() {
        return "pre";
    }
}
```

我们使用pre过滤器类型来处理请求，然后再传递它。

**在过滤器的run()方法中，我们为response_type、scope、client_id和redirect_uri添加查询参数**-我们的授权服务器需要将我们带到其登录页面并返回代码的所有内容。

还要注意shouldFilter()方法，我们只过滤上述3个URI的请求，其他请求不会通过run方法。

## 5. 使用ZuulPost过滤器将代码放入Cookie中

我们计划将代码保存为Cookie，以便我们可以将其发送到授权服务器以获取访问令牌。代码作为请求URL中的查询参数存在，授权服务器在登录后将我们重定向到该URL。

**我们将设置一个Zuul后置过滤器来提取此代码并将其设置在Cookie中，这不仅仅是一个普通的Cookie，而是一个安全、仅HTTP的Cookie，具有非常有限的路径(/auth/token)**：

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    private ObjectMapper mapper = new ObjectMapper();

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        try {
            Map<String, List> params = ctx.getRequestQueryParams();

            if (requestURI.contains("auth/redirect")) {
                Cookie cookie = new Cookie("code", params.get("code").get(0));
                cookie.setHttpOnly(true);
                cookie.setPath(ctx.getRequest().getContextPath() + "/auth/token");
                ctx.getResponse().addCookie(cookie);
            }
        } catch (Exception e) {
            logger.error("Error occured in zuul post filter", e);
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        boolean shouldfilter = false;
        RequestContext ctx = RequestContext.getCurrentContext();
        String URI = ctx.getRequest().getRequestURI();

        if (URI.contains("auth/redirect") || URI.contains("auth/token") || URI.contains("auth/refresh")) {
            shouldfilter = true;
        }
        return shouldfilter;
    }

    @Override
    public int filterOrder() {
        return 10;
    }

    @Override
    public String filterType() {
        return "post";
    }
}
```

为了增加一层额外的保护来抵御CSRF攻击，我们将**在所有Cookie中添加[Same-Site cookie](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)标头**。

为此，我们将创建一个配置类：

```java
@Configuration
public class SameSiteConfig implements WebMvcConfigurer {
    @Bean
    public TomcatContextCustomizer sameSiteCookiesConfig() {
        return context -> {
            final Rfc6265CookieProcessor cookieProcessor = new Rfc6265CookieProcessor();
            cookieProcessor.setSameSiteCookies(SameSiteCookies.STRICT.getValue());
            context.setCookieProcessor(cookieProcessor);
        };
    }
}
```

这里我们将属性设置为strict，以便严格禁止任何跨站点的Cookie传输。

## 6. 从Cookie中获取并使用代码

现在我们在Cookie中有了代码，当前端Angular应用程序尝试触发Token请求时，它将在/auth/token发送请求，因此浏览器当然会发送该Cookie。

因此我们现在在代理中的预过滤器中添加另一个条件，它将**从Cookie中提取代码并将其与其他表单参数一起发送以获取令牌**：

```java
public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    // ...
    else if (requestURI.contains("auth/token"))) {
        try {
            String code = extractCookie(req, "code");
            String formParams = String.format(
                    "grant_type=%s&client_id=%s&client_secret=%s&redirect_uri=%s&code=%s",
                    "authorization_code", CLIENT_ID, CLIENT_SECRET, REDIRECT_URL, code);

            byte[] bytes = formParams.getBytes("UTF-8");
            ctx.setRequest(new CustomHttpServletRequest(req, bytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    // ...
}

private String extractCookie(HttpServletRequest req, String name) {
    Cookie[] cookies = req.getCookies();
    if (cookies != null) {
        for (int i = 0; i < cookies.length; i++) {
            if (cookies[i].getName().equalsIgnoreCase(name)) {
                return cookies[i].getValue();
            }
        }
    }
    return null;
}
```

下面是我们的**CustomHttpServletRequest–用于发送将所需表单参数转换为字节的请求正文**：

```java
public class CustomHttpServletRequest extends HttpServletRequestWrapper {

    private byte[] bytes;

    public CustomHttpServletRequest(HttpServletRequest request, byte[] bytes) {
        super(request);
        this.bytes = bytes;
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        return new ServletInputStreamWrapper(bytes);
    }

    @Override
    public int getContentLength() {
        return bytes.length;
    }

    @Override
    public long getContentLengthLong() {
        return bytes.length;
    }

    @Override
    public String getMethod() {
        return "POST";
    }
}
```

这将在响应中从授权服务器获取访问令牌，接下来，我们将了解如何转换响应。

## 7. 将刷新令牌放入Cookie中

我们在这里计划做的是让客户端获取刷新令牌作为Cookie。

**我们将添加到Zuul后置过滤器中，以从响应的JSON主体中提取刷新令牌并将其设置在Cookie中**。这又是一个安全、仅HTTP的Cookie，具有非常有限的路径(/auth/refresh)：

```java
public Object run() {
    // ...
    else if (requestURI.contains("auth/token") || requestURI.contains("auth/refresh")) {
        InputStream is = ctx.getResponseDataStream();
        String responseBody = IOUtils.toString(is, "UTF-8");
        if (responseBody.contains("refresh_token")) {
            Map<String, Object> responseMap = mapper.readValue(responseBody,
                    new TypeReference<Map<String, Object>>() {});
            String refreshToken = responseMap.get("refresh_token").toString();
            responseMap.remove("refresh_token");
            responseBody = mapper.writeValueAsString(responseMap);

            Cookie cookie = new Cookie("refreshToken", refreshToken);
            cookie.setHttpOnly(true);
            cookie.setPath(ctx.getRequest().getContextPath() + "/auth/refresh");
            cookie.setMaxAge(2592000); // 30 days
            ctx.getResponse().addCookie(cookie);
        }
        ctx.setResponseBody(responseBody);
    }
    // ...
}
```

如我们所见，这里我们在Zuul后置过滤器中添加了一个条件，以读取响应并提取路由auth/token和auth/refresh的刷新令牌。我们对这两者执行完全相同的操作，因为授权服务器在获取访问令牌和刷新令牌时本质上发送了相同的有效负载。

**然后我们从JSON响应中删除了refresh_token，以确保它永远无法在Cookie之外被前端访问**。

这里要注意的另一点是，我们将Cookie的最大期限设置为30天-因为这与Token的过期时间相匹配。

## 8. 从Cookie中获取并使用刷新令牌

现在我们在Cookie中有了刷新令牌，**当前端Angular应用程序尝试触发令牌刷新时，它将在/auth/refresh发送请求**，因此浏览器当然会发送该Cookie。

**因此，我们现在在代理中的预过滤器中添加另一个条件，该条件将从Cookie中提取刷新令牌并将其作为HTTP参数转发**，以便请求有效：

```java
public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    // ...
    else if (requestURI.contains("auth/refresh"))) {
        try {
            String token = extractCookie(req, "token");
            String formParams = String.format(
                    "grant_type=%s&client_id=%s&client_secret=%s&refresh_token=%s",
                    "refresh_token", CLIENT_ID, CLIENT_SECRET, token);

            byte[] bytes = formParams.getBytes("UTF-8");
            ctx.setRequest(new CustomHttpServletRequest(req, bytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    // ...
}
```

这与我们第一次获取访问令牌时所做的类似，但请注意，表单主体有所不同。**现在，我们发送了grant_type为refresh_token而不是authorization_code以及我们之前在Cookie中保存的令牌**。

获得响应后，它会再次在预过滤器中经历与我们在第7节中看到的相同的转换。

## 9. 从Angular刷新访问令牌

最后，让我们修改一下简单的前端应用程序，并实际使用刷新令牌：

这是我们的函数refreshAccessToken()：

```typescript
refreshAccessToken() {
    let headers = new HttpHeaders({
        'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});
    this._http.post('auth/refresh', {}, {headers: headers })
        .subscribe(
            data => this.saveToken(data),
            err => alert('Invalid Credentials')
        );
}
```

请注意我们只是使用现有的saveToken()函数，并向其传递不同的输入。

还要注意，**我们自己没有使用refresh_token添加任何表单参数，因为这将由Zuul过滤器处理**。

## 10. 运行前端

由于我们的前端Angular客户端现在作为Boot应用程序托管，因此运行它会与以前略有不同。

**第一步是相同的，我们需要构建应用程序**：

```shell
mvn clean install
```

这将触发pom.xml中定义的frontend-maven-plugin来构建Angular代码并将UI工件复制到target/classes/static文件夹，此过程将覆盖src/main/resources目录中的任何其他内容。因此，我们需要确保在复制过程中包含此文件夹中的任何必需资源，例如application.yml。

**第二步，我们需要运行SpringBootApplication类UiApplication**，我们的客户端应用程序将在application.yml中指定的端口8089上启动并运行。

## 11. 总结

在本OAuth2教程中，我们学习了如何在Angular客户端应用程序中存储刷新令牌、如何刷新过期的访问令牌以及如何利用Zuul代理完成所有这些操作。