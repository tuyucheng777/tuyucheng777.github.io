---
layout: post
title:  Jersey Bean Validation
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 概述

在本教程中，我们将使用开源框架[Jersey](https://jersey.github.io/)来研究Bean Validation。

正如我们在之前的文章中看到的，**Jersey是一个用于开发RESTful Web服务的开源框架**，我们可以在如何[使用Jersey和Spring创建API](https://www.baeldung.com/jersey-rest-api-with-spring)的介绍中了解有关Jersey的更多详细信息。

## 2. Jersey中的Bean Validation

**校验是核实某些数据是否遵循一个或多个预定义约束的过程**，当然，这是大多数应用程序中非常常见的用例。

[Java Bean Validation](https://beanvalidation.org/)框架(JSR-380)已成为Java中处理此类操作的事实标准，要回顾Java Bean校验的基础知识，请参阅我们之前的[教程](https://www.baeldung.com/javax-validation)。

**Jersey包含一个扩展模块来支持Bean Validation**，要在我们的应用程序中使用此功能，我们首先需要对其进行配置。在下一节中，我们将了解如何配置我们的应用程序。

## 3. 应用程序设置

现在，让我们以[Jersey MVC支持](https://www.baeldung.com/jersey-mvc)文章中的简单Fruit API示例为基础进行构建。

### 3.1 Maven依赖

首先，让我们将Bean Validation依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.glassfish.jersey.ext</groupId>
    <artifactId>jersey-bean-validation</artifactId>
    <version>3.1.1</version>
</dependency>
```

可以从[Maven Central](https://mvnrepository.com/artifact/org.glassfish.jersey.ext/jersey-bean-validation)获取最新版本。

### 3.2 配置服务器

在Jersey中，我们通常在自定义资源配置类中注册我们想要使用的扩展功能。

但是对于Bean校验扩展，则无需进行此注册。**幸运的是，这是Jersey框架自动注册的少数扩展之一**。

最后，为了向客户端发送校验错误，我们将在自定义资源配置中添加一个服务器属性：

```java
public ViewApplicationConfig() {
    packages("cn.tuyucheng.taketoday.jersey.server");
    property(ServerProperties.BV_SEND_ERROR_IN_RESPONSE, true);
}
```

## 4. 校验JAX-RS资源方法

在本节中，我们将解释使用约束注解校验输入参数的两种不同方法：

- 使用内置Bean Validation API约束
- 创建自定义约束和校验器

### 4.1 使用内置约束注解

让我们首先看一下内置的约束注解：

```java
@POST
@Path("/create")
@Consumes(MediaType.APPLICATION_FORM_URLENCODED)
public void createFruit(
        @NotNull(message = "Fruit name must not be null") @FormParam("name") String name,
        @NotNull(message = "Fruit colour must not be null") @FormParam("colour") String colour) {

    Fruit fruit = new Fruit(name, colour);
    SimpleStorageService.storeFruit(fruit);
}
```

在此示例中，我们使用两个表单参数name和colour创建了一个新的Fruit。我们使用@NotNull注解，该注解已是Bean Validation API的一部分。

这对我们的表单参数施加了一个简单的非空约束，**如果其中一个参数为空，则将返回注解中声明的消息**。

当然，我们可以通过单元测试来证明这一点：

```java
@Test
public void givenCreateFruit_whenFormContainsNullParam_thenResponseCodeIsBadRequest() {
    Form form = new Form();
    form.param("name", "apple");
    form.param("colour", null);
    Response response = target("fruit/create").request(MediaType.APPLICATION_FORM_URLENCODED)
            .post(Entity.form(form));

    assertEquals("Http Response should be 400 ", 400, response.getStatus());
    assertThat(response.readEntity(String.class), containsString("Fruit colour must not be null"));
}
```

在上面的例子中，**我们使用JerseyTest支持类来测试Fruit资源**。我们发送一个带有空colour的POST请求，并检查响应是否包含预期的消息。

有关内置校验约束的列表，请查看[文档](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-builtin-constraints)。

### 4.2 定义自定义约束注解

有时我们需要施加更复杂的约束，**这可以通过定义自己的自定义注解来实现**。

使用我们简单的水果API示例，假设我们需要校验所有水果是否都有有效的序列号：

```java
@PUT
@Path("/update")
@Consumes("application/x-www-form-urlencoded")
public void updateFruit(@SerialNumber @FormParam("serial") String serial) {
    //...
}
```

在这个例子中，参数serial必须满足@SerialNumber定义的约束，我们接下来将定义它。

我们首先定义约束注解：

```java
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = { SerialNumber.Validator.class })
public @interface SerialNumber {

    String message() default "Fruit serial number is not valid";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

接下来，我们将定义校验器类SerialNumber.Validator：

```java
public class Validator implements ConstraintValidator<SerialNumber, String> {
    @Override
    public void initialize(SerialNumber serial) {
    }

    @Override
    public boolean isValid(String serial,
                           ConstraintValidatorContext constraintValidatorContext) {

        String serialNumRegex = "^\\d{3}-\\d{3}-\\d{4}$";
        return Pattern.matches(serialNumRegex, serial);
    }
}
```

这里的关键点是Validator类必须实现ConstraintValidator，其中T是我们要校验的值的类型，在我们的例子中是String。

**最后，我们在isValid方法中实现自定义校验逻辑**。

## 5. 资源校验

**此外，Bean Validation API还允许我们使用@Valid注解来校验对象**。

在下一节中，我们将解释使用此注解校验资源类的两种不同方法：

- 请求资源校验
- 响应资源校验

让我们首先向Fruit对象添加@Min注解：

```java
@XmlRootElement
public class Fruit {

    @Min(value = 10, message = "Fruit weight must be 10 or greater")
    private Integer weight;
    // ...
}
```

### 5.1 请求资源校验

首先，我们将在FruitResource类中使用@Valid启用校验：

```java
@POST
@Path("/create")
@Consumes("application/json")
public void createFruit(@Valid Fruit fruit) {
    SimpleStorageService.storeFruit(fruit);
}
```

在上面的例子中，**如果我们尝试创建weight小于10的水果，我们将收到校验错误**。

### 5.2 响应资源校验

同样，在下一个示例中，我们将看到如何校验响应资源：

```java
@GET
@Valid
@Produces("application/json")
@Path("/search/{name}")
public Fruit findFruitByName(@PathParam("name") String name) {
    return SimpleStorageService.findByName(name);
}
```

注意，我们使用相同的@Valid注解，**但这次我们在资源方法级别使用它来确保响应有效**。

## 6. 自定义异常处理程序

在最后一部分中，我们将简要介绍如何创建自定义异常处理程序；**当我们想要在违反特定约束时返回自定义响应时，这很有用**。

让我们首先定义FruitExceptionMapper：

```java
public class FruitExceptionMapper implements ExceptionMapper<ConstraintViolationException> {

    @Override
    public Response toResponse(ConstraintViolationException exception) {
        return Response.status(Response.Status.BAD_REQUEST)
                .entity(prepareMessage(exception))
                .type("text/plain")
                .build();
    }

    private String prepareMessage(ConstraintViolationException exception) {
        StringBuilder message = new StringBuilder();
        for (ConstraintViolation<?> cv : exception.getConstraintViolations()) {
            message.append(cv.getPropertyPath() + " " + cv.getMessage() + "\n");
        }
        return message.toString();
    }
}
```

首先，我们定义一个自定义异常映射提供程序，为了做到这一点，我们使用ConstraintViolationException实现ExceptionMapper接口。

**因此，我们将看到，当抛出此异常时，我们的自定义异常映射器实例的toResponse方法将被调用**。

另外，在这个简单的例子中，我们遍历所有违规行为并追加每个要在响应中发送回的属性和消息。

**接下来，为了使用我们的自定义异常映射器，我们需要注册提供程序**：

```java
@Override
protected Application configure() {
    ViewApplicationConfig config = new ViewApplicationConfig();
    config.register(FruitExceptionMapper.class);
    return config;
}
```

最后，我们添加一个端点来返回无效的水果，以显示异常处理程序的运行：

```java
@GET
@Produces(MediaType.TEXT_HTML)
@Path("/exception")
@Valid
public Fruit exception() {
    Fruit fruit = new Fruit();
    fruit.setName("a");
    fruit.setColour("b");
    return fruit;
}
```

## 7. 总结

在本教程中，我们探索了Jersey Bean Validation API扩展。

首先，我们介绍了如何在Jersey中使用Bean Validation API，此外，我们还了解了如何配置示例Web应用程序。

最后，我们研究了使用Jersey进行校验的几种方法以及如何编写自定义异常处理程序。