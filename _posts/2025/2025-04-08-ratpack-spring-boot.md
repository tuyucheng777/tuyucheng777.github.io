---
layout: post
title:  Ratpack与Spring Boot的集成
category: webmodules
copyright: webmodules
excerpt: Ratpack
---

## 1. 概述

之前，我们介绍了[Ratpack](https://www.baeldung.com/ratpack)以及它[与Google Guice的集成](https://www.baeldung.com/ratpack-google-guice)。

在这篇简短的文章中，我们将展示如何将Ratpack与Spring Boot集成。

## 2. Maven依赖

在继续之前，让我们在pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-spring-boot-starter</artifactId>
    <version>1.4.6</version>
    <type>pom</type>
</dependency>
```

[ratpack-spring-boot-starter](https://mvnrepository.com/search?q=ratpack-spring-boot-starter) pom依赖会自动将[ratpack-spring-boot](https://mvnrepository.com/search?q=ratpack-spring-boot)和[spring-boot-starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter)添加到我们的依赖中。

## 3. 将Ratpack与Spring Boot集成

我们可以将Ratpack嵌入到Spring Boot中，作为Tomcat或Undertow提供的Servlet容器的替代方案，我们只需要用[@EnableRatpack](https://ratpack.io/manual/current/api/ratpack/spring/config/EnableRatpack.html)标注一个Spring配置类，并声明[Action](https://ratpack.io/manual/current/api/ratpack/func/Action.html)[<Chain\>](https://ratpack.io/manual/current/api/ratpack/core/handling/Chain.html)类型的Bean：

```java
@SpringBootApplication
@EnableRatpack
public class EmbedRatpackApp {

    @Autowired
    private Content content;

    @Autowired
    private ArticleList list;

    @Bean
    public Action<Chain> home() {
        return chain -> chain.get(ctx -> ctx.render(content.body()));
    }

    public static void main(String[] args) {
        SpringApplication.run(EmbedRatpackApp.class, args);
    }
}
```

对于更熟悉Spring Boot的人来说，Action<Chain\>可以充当Web过滤器和/或控制器。

在提供静态文件时，Ratpack会在@Autowired [ChainConfigurers](https://github.com/ratpack/ratpack/blob/master/ratpack-spring-boot/src/main/java/ratpack/spring/config/internal/ChainConfigurers.java#L63)中自动为/public和/static下的静态资源注册处理程序。

但是，[目前](https://github.com/ratpack/ratpack/blob/master/ratpack-spring-boot/src/main/java/ratpack/spring/config/RatpackProperties.java#L202)这个“魔法”的实现依赖于我们的项目设置和开发环境。因此，如果我们要实现在不同环境中稳定提供静态资源，我们应该明确指定baseDir：

```java
@Bean
public ServerConfig ratpackServerConfig() {
    return ServerConfig
        .builder()
        .findBaseDir("static")
        .build();
}
```

上述代码假设我们在classpath下有一个static文件夹；此外，我们将[ServerConfig](https://ratpack.io/manual/current/api/ratpack/core/server/ServerConfig.html) Bean命名为ratpackServerConfig以覆盖[RatpackConfiguration](https://github.com/ratpack/ratpack/blob/master/ratpack-spring-boot/src/main/java/ratpack/spring/config/RatpackConfiguration.java#L70)中注册的默认Bean。

然后我们可以使用[ratpack-test](https://mvnrepository.com/artifact/io.ratpack/ratpack-test)测试我们的应用程序：

```java
MainClassApplicationUnderTest appUnderTest = new MainClassApplicationUnderTest(EmbedRatpackApp.class);

@Test
public void whenSayHello_thenGotWelcomeMessage() {
    assertEquals("hello tuyucheng!", appUnderTest
        .getHttpClient()
        .getText("/hello"));
}

@Test
public void whenRequestList_thenGotArticles()  {
    assertEquals(3, appUnderTest
        .getHttpClient()
        .getText("/list")
        .split(",").length);
}

@Test
public void whenRequestStaticResource_thenGotStaticContent() {
    assertThat(appUnderTest
        .getHttpClient()
        .getText("/"), containsString("page is static"));
}
```

## 4. 将Spring Boot与Ratpack集成

首先，我们将在Spring配置类中注册所需的Bean：

```java
@Configuration
public class Config {

    @Bean
    public Content content() {
        return () -> "hello tuyucheng!";
    }
}
```

然后我们可以使用ratpack-spring-boot提供的便捷方法[spring(...)](https://ratpack.io/manual/current/api/ratpack/spring/Spring.html#spring-java.lang.Class-java.lang.String...-)轻松创建一个注册器：

```java
public class EmbedSpringBootApp {

    public static void main(String[] args) throws Exception {
        RatpackServer.start(server -> server
            .registry(spring(Config.class))
            .handlers(chain -> chain.get(ctx -> ctx.render(ctx
                .get(Content.class)
                .body()))));
    }
}
```

在Ratpack中，注册器用于请求处理过程中的处理程序间通信，我们在处理程序中使用的[Context](https://ratpack.io/manual/current/api/ratpack/core/handling/Context.html)对象实现了[Registry](https://ratpack.io/manual/current/api/ratpack/exec/registry/Registry.html)接口。

这里，Spring Boot提供的[ListableBeanFactory](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/ListableBeanFactory.html)实例适配到Registry，以支持Handler的Context中的注册器。因此，当我们想从Context中获取特定类型的对象时，将使用ListableBeanFactory来查找匹配的Bean。

## 5. 总结

在本教程中，我们探讨了如何集成Spring Boot和Ratpack。