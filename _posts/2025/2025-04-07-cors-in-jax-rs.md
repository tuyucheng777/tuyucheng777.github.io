---
layout: post
title:  JAX-RS中的CORS
category: webmodules
copyright: webmodules
excerpt: RESTEasy
---

## 1. 概述

在这篇简短的文章中，我们将学习如何在基于JAX-RS的系统中启用[CORS](https://www.baeldung.com/cs/cors-preflight-requests)(跨源资源共享)，我们将在JAX-RS之上设置一个应用程序以启用CORS机制。

## 2. 如何启用CORS机制

我们可以通过两种方式在JAX-RS中启用CORS；第一种也是最基本的方法是创建一个过滤器，以便在每次请求时在运行时注入必要的响应标头。另一种方法是在每个URL端点中手动添加适当的标头。

理想情况下，应该使用第一个解决方案；但是，当这不是一个可能时，更手动的选项在技术上也是可行的。

### 2.1 使用过滤器

JAX-RS具有[ContainerResponseFilter](https://docs.oracle.com/javaee/7/api/javax/ws/rs/container/ContainerResponseFilter.html)接口-由容器响应过滤器实现。通常，此过滤器实例全局应用于任何HTTP响应。

我们将实现此接口来创建一个自定义过滤器，它将向每个传出请求注入Access-Control-Allow-\*标头并启用CORS机制：

```java
@Provider
public class CorsFilter implements ContainerResponseFilter {

    @Override
    public void filter(ContainerRequestContext requestContext,
                       ContainerResponseContext responseContext) throws IOException {
        responseContext.getHeaders().add(
                "Access-Control-Allow-Origin", "*");
        responseContext.getHeaders().add(
                "Access-Control-Allow-Credentials", "true");
        responseContext.getHeaders().add(
                "Access-Control-Allow-Headers",
                "origin, content-type, accept, authorization");
        responseContext.getHeaders().add(
                "Access-Control-Allow-Methods",
                "GET, POST, PUT, DELETE, OPTIONS, HEAD");
    }
}
```

这里有几点：

- 实现ContainerResponseFilter的过滤器必须使用@Provider明确标注，才能被JAX-RS运行时发现
- 我们在“Access-Control-Allow-\*”标头中注入了“\*”，这意味着可以通过任何域访问此服务器实例的任何URL端点；如果我们想明确限制跨域访问，我们必须在此标头中提及该域

### 2.2 使用Header修改每个端点

如前所述，我们也可以在端点级别明确注入“Access-Control-Allow-\*”标头：

```java
@GET
@Path("/")
@Produces({MediaType.TEXT_PLAIN})
public Response index() {
    return Response
            .status(200)
            .header("Access-Control-Allow-Origin", "*")
            .header("Access-Control-Allow-Credentials", "true")
            .header("Access-Control-Allow-Headers",
                    "origin, content-type, accept, authorization")
            .header("Access-Control-Allow-Methods",
                    "GET, POST, PUT, DELETE, OPTIONS, HEAD")
            .entity("")
            .build();
}
```

这里要注意的一点是，如果我们尝试在大型应用程序中启用CORS，不应该使用这种方法，因为在这种情况下，我们必须手动将标头注入每个URL端点，这将引入额外的开销。

但是，这种技术可以在只需要在某些URL端点中启用CORS的应用程序中使用。

## 3. 测试

应用程序启动后，我们可以使用curl命令测试标头，示例标头输出应如下所示：

```text
HTTP/1.1 200 OK
Date : Tue, 13 May 2014 12:30:00 GMT
Connection : keep-alive
Access-Control-Allow-Origin : *
Access-Control-Allow-Credentials : true
Access-Control-Allow-Headers : origin, content-type, accept, authorization
Access-Control-Allow-Methods : GET, POST, PUT, DELETE, OPTIONS, HEAD
Transfer-Encoding : chunked
```

此外，我们可以创建一个简单的AJAX函数并检查跨域功能：

```javascript
function call(url, type, data) {
    var request = $.ajax({
        url: url,
        method: "GET",
        data: (data) ? JSON.stringify(data) : "",
        dataType: type
    });

    request.done(function(resp) {
        console.log(resp);
    });

    request.fail(function(jqXHR, textStatus) {
        console.log("Request failed: " + textStatus);
    });
};
```

当然，为了真正执行检查，我们必须在与我们正在使用的API不同的来源上运行它。

你可以通过在单独的端口上运行客户端应用程序轻松地在本地执行此操作-**因为端口也决定了来源**。

## 4. 总结

在本文中，我们展示了如何在基于JAX-RS的应用程序中实现CORS机制。
