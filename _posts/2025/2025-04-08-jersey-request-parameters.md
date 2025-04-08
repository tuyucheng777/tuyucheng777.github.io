---
layout: post
title:  Jersey请求参数
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 简介

[Jersey](https://www.baeldung.com/jersey-jax-rs-client)是一个用于创建RESTful Web服务的流行Java框架。

在本教程中，我们将探讨如何通过一个简单的Jersey项目读取不同的请求参数类型。

## 2. 项目设置

使用Maven原型，我们将能够为我们的文章生成一个示例项目：

```shell
mvn archetype:generate -DarchetypeArtifactId=jersey-quickstart-grizzly2
  -DarchetypeGroupId=org.glassfish.jersey.archetypes -DinteractiveMode=false
  -DgroupId=com.example -DartifactId=simple-service -Dpackage=com.example
  -DarchetypeVersion=2.28
```

**生成的Jersey项目将在Grizzly容器上运行**。

现在，默认情况下，我们的应用程序的端点将是http://localhost:8080/myapp。

让我们添加一个items资源，我们将用它来进行实验：

```java
@Path("items")
public class ItemsController {
    // our endpoints are defined here
}
```

顺便说一句，请注意，[Jersey也能与Spring控制器很好地配合使用](https://www.baeldung.com/jersey-rest-api-with-spring)。

## 3. 标注参数类型

因此，在实际读取任何请求参数之前，让我们先澄清一些规则，**允许的参数类型是**：

- 原始类型，例如float和char
- 具有带有单个String参数的构造函数的类型
- 具有fromString或valueOf静态方法的类型；对于这些类型，必须有一个String参数
- 上述类型的集合(例如List、Set和SortedSet)

另外，我们可以注册ParamConverterProvider JAX-RS扩展SPI的实现，返回类型必须是能够从字符串转换为类型的ParamConverter实例。

## 4. Cookie

我们可以使用@CookieParam注解来解析Jersey方法中的[cookie值](https://www.baeldung.com/cookies-java)：

```java
@GET
public String jsessionid(@CookieParam("JSESSIONId") String jsessionId) {
    return "Cookie parameter value is [" + jsessionId+ "]";
}
```

如果我们启动容器，我们可以通过[cURL](https://www.baeldung.com/curl-rest)来查看响应：

```shell
> curl --cookie "JSESSIONID=5BDA743FEBD1BAEFED12ECE124330923" http://localhost:8080/myapp/items
Cookie parameter value is [5BDA743FEBD1BAEFED12ECE124330923]
```

## 5. 标头

或者，我们可以使用@HeaderParam注解解析[HTTP标头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)：

```java
@GET
public String contentType(@HeaderParam("Content-Type") String contentType) {
    return "Header parameter value is [" + contentType+ "]";
}
```

我们再测试一下：

```shell
> curl --header "Content-Type: text/html" http://localhost:8080/myapp/items
Header parameter value is [text/html]
```

## 6. 路径参数

尤其是对于RESTful API，在路径中包含信息是很常见的。

我们可以使用@PathParam提取路径元素：

```java
@GET
@Path("/{id}")
public String itemId(@PathParam("id") Integer id) {
    return "Path parameter value is [" + id + "]";
}
```

让我们发送另一个带有值3的curl命令：

```shell
> curl http://localhost:8080/myapp/items/3
Path parameter value is [3]
```

## 7. 查询参数

我们通常在RESTful API中使用查询参数来获取可选信息。

要读取这些值，我们可以使用@QueryParam注解：

```java
@GET
public String itemName(@QueryParam("name") String name) {
    return "Query parameter value is [" + name + "]";
}
```

现在我们可以像以前一样使用curl进行测试：

```shell
> curl http://localhost:8080/myapp/items?name=Toaster
Query parameter value if [Toaster]
```

## 8. 表单参数

为了从表单提交中读取参数，我们将使用@FormParam注解：

```java
@POST
public String itemShipment(@FormParam("deliveryAddress") String deliveryAddress, @FormParam("quantity") Long quantity) {
    return "Form parameters are [deliveryAddress=" + deliveryAddress+ ", quantity=" + quantity + "]";
}
```

我们还需要设置适当的Content-Type来模拟表单提交操作，让我们使用-d标志设置表单参数：

```shell
> curl -X POST -H 'Content-Type:application/x-www-form-urlencoded' \
  -d 'deliveryAddress=Washington nr 4&quantity=5' \
  http://localhost:8080/myapp/items
Form parameters are [deliveryAddress=Washington nr 4, quantity=5]
```

## 9. 矩阵参数

**[矩阵参数](https://www.baeldung.com/spring-mvc-matrix-variables)是一种更灵活的查询参数，因为它们可以在URL中的任何位置添加**。

例如，在http://localhost:8080/myapp;name=value/items中，矩阵参数是name。

要读取这些值，我们可以使用可用的@MatrixParam注解：

```java
@GET
public String itemColors(@MatrixParam("colors") List<String> colors) {
    return "Matrix parameter values are " + Arrays.toString(colors.toArray());
}
```

现在再次测试我们的端点：

```shell
> curl http://localhost:8080/myapp/items;colors=blue,red
Matrix parameter values are [blue,red]
```

## 10. Bean参数

最后，我们将检查如何使用Bean参数组合请求参数；需要澄清的是，[Bean参数](https://www.baeldung.com/jersey-bean-validation)实际上是组合不同类型的请求参数的对象。

我们将在这里使用一个标头参数，一个路径和一个表单参数：

```java
public class ItemOrder {
    @HeaderParam("coupon")
    private String coupon;

    @PathParam("itemId")
    private Long itemId;

    @FormParam("total")
    private Double total;

    //getter and setter

    @Override
    public String toString() {
        return "ItemOrder {coupon=" + coupon + ", itemId=" + itemId + ", total=" + total + '}';
    }
}
```

另外，为了获得这样的参数组合，我们将使用@BeanParam注解：

```java
@POST
@Path("/{itemId}")
public String itemOrder(@BeanParam ItemOrder itemOrder) {
    return itemOrder.toString();
}
```

在curl命令中，我们添加了这三种类型的参数，最终得到一个ItemOrder对象：

```shell
> curl -X POST -H 'Content-Type:application/x-www-form-urlencoded' \
  --header 'coupon:FREE10p' \
  -d total=70 \
  http://localhost:8080/myapp/items/28711
ItemOrder {coupon=FREE10p, itemId=28711, total=70}
```

## 11. 总结

总而言之，我们为Jersey项目创建了一个简单的设置，以帮助我们探索如何使用Jersey从请求中读取不同的参数。