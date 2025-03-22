---
layout: post
title:  在Spring REST中的@RequestBody中使用XML
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

虽然[JSON](https://www.baeldung.com/java-json)是[RESTful](https://www.baeldung.com/cs/rest-architecture)服务的事实标准，但在某些情况下，我们可能希望使用[XML](https://www.baeldung.com/java-xml)。我们可以出于不同的原因回退到XML：遗留应用程序、使用更详细的格式、标准化模式等。

[Spring](https://www.baeldung.com/spring-tutorial)为我们提供了一种支持XML端点的简单方法，而无需我们进行任何工作。在本教程中，我们将学习如何利用[Jackson](https://www.baeldung.com/jackson) XML来解决此问题。

## 2. 依赖

第一步是添加[依赖](https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-xml)以允许XML映射。即使我们使用spring-boot-starter-web，它默认也不包含XML支持的库：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.16.0</version>
</dependency>
```

**我们可以通过省略版本来利用Spring Boot版本管理系统，确保在所有依赖项中使用正确的Jackson库版本**。

或者，我们可以使用JAXB来做同样的事情，但总的来说，它更冗长，而且Jackson通常为我们提供了更好的API。但是，如果我们使用Java 8，[JAXB](https://www.baeldung.com/jaxb)库位于javax与实现一起打包，我们不需要向我们的应用程序添加任何其他依赖项。

**在从9开始的Java版本中，javax包已移动并重命名为jakarta，因此JAXB需要[额外依赖](https://mvnrepository.com/artifact/jakarta.xml.bind/jakarta.xml.bind-api/4.0.0)**：

```xml
<dependency>
    <groupId>jakarta.xml.bind</groupId>
    <artifactId>jakarta.xml.bind-api</artifactId>
    <version>4.0.0</version>
</dependency>
```

此外，它需要XML映射器的运行时实现，这可能会造成太多混乱和微妙的问题。

## 3. 端点

由于JSON是Spring REST[控制器](https://www.baeldung.com/spring-controllers)的默认格式，因此我们需要显式标识使用和生成XML的端点。让我们考虑这个简单的Echo控制器：

```java
@RestController
@RequestMapping("/users")
public class UserEchoController {
    @ResponseStatus(HttpStatus.CREATED)
    @PostMapping(consumes = MediaType.APPLICATION_JSON_VALUE, produces = MediaType.APPLICATION_JSON_VALUE)
    public User echoJsonUser(@RequestBody User user) {
        return user;
    }

    @ResponseStatus(HttpStatus.CREATED)
    @PostMapping(consumes = MediaType.APPLICATION_XML_VALUE, produces = MediaType.APPLICATION_XML_VALUE)
    public User echoXmlUser(@RequestBody User user) {
        return user;
    }
}
```

控制器的唯一目的是接收用户并将其发回。这两个端点之间的唯一区别是第一个端点适用于JSON格式，我们在[@PostMapping](https://www.baeldung.com/spring-new-requestmapping-shortcuts#new-annotations)中显式指定它，但对于JSON，我们可以省略[consumes和produces](https://www.baeldung.com/spring-boot-json#create)属性。

第二个端点使用XML，**我们必须通过向consumes和produces提供正确的类型来明确识别它**，这是我们配置端点所需要做的唯一事情。

## 4. 映射

我们将使用以下User类：

```java
public class User {
    private Long id;
    private String firstName;
    private String secondName;

    public User() {
    }

    // getters, setters, equals, hashCode
}
```

从技术上讲，我们不需要任何其他东西，端点应该立即支持以下XML：

```xml
<User>
    <id>1</id>
    <firstName>John</firstName>
    <secondName>Doe</secondName>
</User>
```

**但是，如果我们想要提供其他名称或将旧约定转换为我们在应用程序中使用的约定，我们可能需要使用[特殊注解](https://www.baeldung.com/jackson-xml-serialization-and-deserialization)**，@JacksonXmlRootElement和@JacksonXmlProperty是用于此目的的最常见注解。

如果我们选择使用JAXB，也可以仅使用注解来配置映射，并且有一组不同的[注解](https://www.baeldung.com/java-xml-libraries#JAXB)，例如，@XmlRootElement和@XmlAttribute。总的来说，这个过程非常相似。但是，请注意，JAXB可能需要显式映射。

## 5. 总结

Spring REST为我们提供了一种创建RESTful服务的便捷方式，并且它们不仅仅局限于JSON，我们可以将它们与其他格式一起使用，例如XML。**总体而言，过渡是透明的，并且整个设置都是通过几个策略性放置的注解完成的**。