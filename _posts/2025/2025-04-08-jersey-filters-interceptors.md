---
layout: post
title:  Jersey过滤器和拦截器
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 简介

在本文中，我们将解释过滤器和拦截器在Jersey框架中的工作原理，以及它们之间的主要区别。

我们将在这里使用Jersey 3，并使用Tomcat 10服务器测试我们的应用程序。

## 2. 应用程序设置

让我们首先在服务器上创建一个简单的资源：

```java
@Path("/greetings")
public class Greetings {
    @GET
    public String getHelloGreeting() {
        return "hello";
    }
}
```

另外，让我们为应用程序创建相应的服务器配置：

```java
@ApplicationPath("/*")
public class ServerConfig extends ResourceConfig {

    public ServerConfig() {
        packages("cn.tuyucheng.taketoday.jersey.server");
    }
}
```

如果你想深入了解如何使用Jersey创建API，可以查看[这篇文章](https://www.baeldung.com/jersey-rest-api-with-spring)。

## 3. 过滤器

现在，让我们开始使用过滤器。

简单来说，**过滤器让我们可以修改请求和响应的属性**，例如HTTP标头。过滤器既可以应用于服务器端，也可以应用于客户端。

请记住，**无论是否找到资源，过滤器都会始终执行**。

### 3.1 实现请求服务器过滤器

让我们从服务器端的过滤器开始，创建一个请求过滤器。

**我们将通过实现ContainerRequestFilter接口并将其注册为服务器中的提供程序来实现这一点**：

```java
@Provider
public class RestrictedOperationsRequestFilter implements ContainerRequestFilter {

    @Override
    public void filter(ContainerRequestContext ctx) throws IOException {
        if (ctx.getLanguage() != null && "EN".equals(ctx.getLanguage()
                .getLanguage())) {

            ctx.abortWith(Response.status(Response.Status.FORBIDDEN)
                    .entity("Cannot access")
                    .build());
        }
    }
}
```

这个简单的过滤器只是通过调用abortWith()方法来拒绝请求中带有语言“EN”的请求。

如示例所示，我们只需实现一种接收请求上下文的方法，我们可以根据需要对其进行修改。

请记住，**此过滤器是在资源匹配后执行的**。

如果我们想要在资源匹配之前执行过滤器，**我们可以使用预匹配过滤器，通过使用@PreMatching注解来标注我们的过滤器**：

```java
@Provider
@PreMatching
public class PrematchingRequestFilter implements ContainerRequestFilter {

    @Override
    public void filter(ContainerRequestContext ctx) throws IOException {
        if (ctx.getMethod().equals("DELETE")) {
            LOG.info("\"Deleting request");
        }
    }
}
```

如果现在尝试访问我们的资源，我们可以检查预匹配过滤器是否首先执行：

```text
2018-02-25 16:07:27,800 [http-nio-8080-exec-3] INFO  c.t.t.j.s.f.PrematchingRequestFilter - prematching filter
2018-02-25 16:07:27,816 [http-nio-8080-exec-3] INFO  c.t.t.j.s.f.RestrictedOperationsRequestFilter - Restricted operations filter
```

### 3.2 实现响应服务器过滤器

我们现在将在服务器端实现一个响应过滤器，它只会向响应添加一个新标头。

为此，**我们的过滤器必须实现ContainerResponseFilter接口并实现其唯一方法**：

```java
@Provider
public class ResponseServerFilter implements ContainerResponseFilter {

    @Override
    public void filter(ContainerRequestContext requestContext,
                       ContainerResponseContext responseContext) throws IOException {
        responseContext.getHeaders().add("X-Test", "Filter test");
    }
}
```

请注意，ContainerRequestContext参数仅用作只读-因为我们在处理响应。

### 3.3 实现客户端过滤器

现在我们将使用客户端过滤器，这些过滤器的工作方式与服务器过滤器相同，我们必须实现的接口与服务器端的接口非常相似。

让我们通过向请求添加属性的过滤器来观察它的实际效果：

```java
@Provider
public class RequestClientFilter implements ClientRequestFilter {

    @Override
    public void filter(ClientRequestContext requestContext) throws IOException {
        requestContext.setProperty("test", "test client request filter");
    }
}
```

我们还创建一个Jersey客户端来测试这个过滤器：

```java
public class JerseyClient {

    private static String URI_GREETINGS = "http://localhost:8080/jersey/greetings";

    public static String getHelloGreeting() {
        return createClient().target(URI_GREETINGS)
                .request()
                .get(String.class);
    }

    private static Client createClient() {
        ClientConfig config = new ClientConfig();
        config.register(RequestClientFilter.class);

        return ClientBuilder.newClient(config);
    }
}
```

请注意，我们必须将过滤器添加到客户端配置中才能注册它。

最后，我们还将为客户端中的响应创建一个过滤器。

其工作方式与服务器中的非常相似，但实现了ClientResponseFilter接口：

```java
@Provider
public class ResponseClientFilter implements ClientResponseFilter {

    @Override
    public void filter(ClientRequestContext requestContext, ClientResponseContext responseContext) throws IOException {
        responseContext.getHeaders().add("X-Test-Client", "Test response client filter");
    }
}
```

再次强调，ClientRequestContext是只读的。

## 4. 拦截器

拦截器与请求和响应中包含的HTTP消息体的编组和解组联系更紧密，它们既可以在服务器端使用，也可以在客户端使用。

请记住，**它们是在过滤器之后执行的，并且仅当存在消息正文时才会执行**。

拦截器有两种类型：ReaderInterceptor和WriterInterceptor，它们对于服务器端和客户端都是相同的。

接下来，我们将在服务器上创建另一个资源-该资源通过POST访问并在正文中接收参数，因此在访问它时将执行拦截器：

```java
@POST
@Path("/custom")
public Response getCustomGreeting(String name) {
    return Response.status(Status.OK.getStatusCode())
        .build();
}
```

我们还将向我们的Jersey客户端添加一种新方法-来测试这一新资源：

```java
public static Response getCustomGreeting() {
    return createClient().target(URI_GREETINGS + "/custom")
        .request()
        .post(Entity.text("custom"));
}
```

### 4.1 实现ReaderInterceptor

ReaderInterceptor允许我们操纵入站流，因此我们可以使用它们来修改服务器端的请求或客户端的响应。

让我们在服务器端创建一个拦截器，以便在拦截的请求正文中写入自定义消息：

```java
@Provider
public class RequestServerReaderInterceptor implements ReaderInterceptor {

    @Override
    public Object aroundReadFrom(ReaderInterceptorContext context)
            throws IOException, WebApplicationException {
        InputStream is = context.getInputStream();
        String body = new BufferedReader(new InputStreamReader(is)).lines()
                .collect(Collectors.joining("\n"));

        context.setInputStream(new ByteArrayInputStream(
                (body + " message added in server reader interceptor").getBytes()));

        return context.proceed();
    }
}
```

注意，我们**必须调用proceed()方法来调用链中的下一个拦截器**；一旦所有拦截器都执行完毕，就会调用相应的消息体读取器。

### 4.2 实现WriterInterceptor

WriterInterceptor的工作方式与ReaderInterceptor非常相似，但它们操纵出站流-以便我们可以在客户端的请求中或在服务器端的响应中使用它。

让我们创建一个WriterInterceptor，在客户端向请求添加一条消息：

```java
@Provider
public class RequestClientWriterInterceptor implements WriterInterceptor {

    @Override
    public void aroundWriteTo(WriterInterceptorContext context) throws IOException, WebApplicationException {
        context.getOutputStream()
                .write(("Message added in the writer interceptor in the client side").getBytes());

        context.proceed();
    }
}
```

再次，我们必须调用方法proceed()来调用下一个拦截器。

当所有拦截器都被执行时，适当的消息体写入器将被调用。

**不要忘记，你必须在客户端配置中注册此拦截器**，就像我们之前对客户端过滤器所做的那样：

```java
private static Client createClient() {
    ClientConfig config = new ClientConfig();
    config.register(RequestClientFilter.class);
    config.register(RequestWriterInterceptor.class);

    return ClientBuilder.newClient(config);
}
```

## 5. 执行顺序

让我们用一张图来总结一下目前所见的所有内容，该图显示了在客户端向服务器发出请求期间过滤器和拦截器的执行时间：

![](/assets/images/2025/webmodules/jerseyfiltersinterceptors01.png)

我们可以看到，**过滤器总是首先执行，而拦截器则在调用相应的消息体读取器或写入器之前执行**。

如果看一下我们创建的过滤器和拦截器，它们将按照以下顺序执行：

1. RequestClientFilter
2. RequestClientWriterInterceptor
3. PrematchingRequestFilter
4. RestrictedOperationsRequestFilter
5. RequestServerReaderInterceptor
6. ResponseServerFilter
7. ResponseClientFilter

此外，当我们有多个过滤器或拦截器时，我们可以通过使用@Priority注解来指定确切的执行顺序。

优先级用整数指定，并按请求的升序对过滤器和拦截器进行排序，按响应的降序对过滤器和拦截器进行排序。

让我们为RestrictedOperationsRequestFilter添加优先级：

```java
@Provider
@Priority(Priorities.AUTHORIZATION)
public class RestrictedOperationsRequestFilter implements ContainerRequestFilter {
    // ...
}
```

请注意，我们出于授权目的使用了预定义的优先级。

## 6. 名称绑定

到目前为止我们所看到的过滤器和拦截器被称为全局的，因为它们会针对每个请求和响应执行。

但是，**它们也可以被定义为仅针对特定的资源方法执行**，这称为名称绑定。

### 6.1 静态绑定

执行名称绑定的一种方法是静态地创建将在所需资源中使用的特定注解，此注解必须包含@NameBinding元注解。

让我们在应用程序中创建一个：

```java
@NameBinding
@Retention(RetentionPolicy.RUNTIME)
public @interface HelloBinding {
}
```

之后，我们可以用这个@HelloBinding注解来标注一些资源：

```java
@GET
@HelloBinding
public String getHelloGreeting() {
    return "hello";
}
```

最后，我们也将用这个注解来标注我们的一个过滤器，因此这个过滤器将只对访问getHelloGreeting()方法的请求和响应执行：

```java
@Provider
@Priority(Priorities.AUTHORIZATION)
@HelloBinding
public class RestrictedOperationsRequestFilter implements ContainerRequestFilter {
    // ...
}
```

请记住，我们的RestrictedOperationsRequestFilter将不再针对其余资源触发。

### 6.2 动态绑定

另一种方法是使用动态绑定，它在启动期间加载到配置中。

让我们首先为这一部分向我们的服务器添加另一个资源：

```java
@GET
@Path("/hi")
public String getHiGreeting() {
    return "hi";
}
```

现在，让我们通过实现DynamicFeature接口来为该资源创建一个绑定：

```java
@Provider
public class HelloDynamicBinding implements DynamicFeature {

    @Override
    public void configure(ResourceInfo resourceInfo, FeatureContext context) {
        if (Greetings.class.equals(resourceInfo.getResourceClass())
                && resourceInfo.getResourceMethod().getName().contains("HiGreeting")) {
            context.register(ResponseServerFilter.class);
        }
    }
}
```

在本例中，我们将getHiGreeting()方法与之前创建的ResponseServerFilter关联起来。

重要的是要记住，我们必须从这个过滤器中删除@Provider注解，因为我们现在通过DynamicFeature对其进行配置。

如果不这样做，过滤器将被执行两次：一次作为全局过滤器，另一次作为绑定到getHiGreeting()方法的过滤器。

## 7. 总结

在本教程中，我们重点了解过滤器和拦截器在Jersey 3中的工作方式以及如何在Web应用程序中使用它们。