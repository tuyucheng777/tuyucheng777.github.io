---
layout: post
title:  RESTEasy指南
category: webmodules
copyright: webmodules
excerpt: RESTEasy
---

## 1. 简介

JAX-RS(Java API for RESTful Web Services)是一组Java API，提供对创建REST API的支持，并且该框架充分利用了注解来简化这些API的开发和部署。

在本教程中，我们将使用RESTEasy(JBoss提供的JAX-RS规范的可移植实现)，以创建简单的RESTful Web服务。

## 2. 项目设置

我们将考虑两种可能的情况：

- 独立安装：适用于每个应用服务器
- JBoss AS设置：仅考虑在JBoss AS中部署

### 2.1 独立设置

让我们从使用独立设置的**JBoss WildFly 10**开始。

JBoss WildFly 10附带RESTEasy版本6.2.9.Final，但正如你所见，我们将使用新的6.2.9.Final版本配置pom.xml。

并且得益于**resteasy-servlet-initializer**，RESTEasy通过ServletContainerInitializer集成接口提供与独立Servlet 3.0容器的集成。

让我们看一下pom.xml：


```xml
<properties>
    <resteasy.version>6.2.9.Final</resteasy.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-servlet-initializer</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-client</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
</dependencies>
```

jboss-deployment-structure.xml

在JBoss中，以WAR、JAR或EAR形式部署的所有内容都是模块，这些模块称为动态模块。

除此之外，$JBOSS_HOME/modules中还有一些静态模块。由于JBoss有RESTEasy静态模块-对于独立部署，jboss-deployment-structure.xml是必需的，以便排除其中一些模块。

这样，我们的WAR中包含的所有类和JAR文件都将被加载：

```xml
<jboss-deployment-structure>
    <deployment>
        <exclude-subsystems>
            <subsystem name="resteasy" />
        </exclude-subsystems>
        <exclusions>
            <module name="javaee.api" />
            <module name="jakarta.ws.rs.api"/>
            <module name="org.jboss.resteasy.resteasy-jaxrs" />
        </exclusions>
        <local-last value="true" />
    </deployment>
</jboss-deployment-structure>
```

### 2.2 JBoss设置

如果你要使用JBoss 6或更高版本运行RESTEasy，你可以选择采用应用程序服务器中已经捆绑的库，从而简化pom：

```xml
<dependencies>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-jaxrs</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
</dependencies>
```

请注意，jboss-deployment-structure.xml不再需要。

## 3. 服务器端代码

### 3.1 Servlet版本3 web.xml

现在让我们快速浏览一下我们简单项目的web.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
     http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

    <display-name>RestEasy Example</display-name>

    <context-param>
        <param-name>resteasy.servlet.mapping.prefix</param-name>
        <param-value>/rest</param-value>
    </context-param>
</web-app>
```

仅当你想在API应用程序前面添加相对路径时才需要**resteasy.servlet.mapping.prefix**。

此时，需要特别注意的是，我们没有在web.xml中声明任何Servlet，因为resteasy-servlet-initializer已作为依赖添加到pom.xml中。原因是-RESTEasy提供了实现jakarta.server.ServletContainerInitializer的org.jboss.resteasy.plugins.servlet.ResteasyServletInitializer类。

ServletContainerInitializer是一个初始化程序，它在任何Servlet上下文准备好之前执行-你可以使用此初始化程序为你的应用程序定义Servlet、过滤器或监听器。

### 3.2 Application类

jakarta.ws.rs.core.Application类是一个标准JAX-RS类，你可以实现它来提供有关部署的信息：

```java
@ApplicationPath("/rest")
public class RestEasyServices extends Application {

    private Set<Object> singletons = new HashSet<Object>();

    public RestEasyServices() {
        singletons.add(new MovieCrudService());
    }

    @Override
    public Set<Object> getSingletons() {
        return singletons;
    }
}
```

如你所见，这只是一个列出所有JAX-RS根资源和提供程序的类，并使用@ApplicationPath注解进行标注。

如果返回类和单例的任何空集，则将扫描WAR以查找JAX-RS注解资源和提供程序类。

### 3.3 服务实现类

最后，让我们在这里看一个实际的API定义：

```java
@Path("/movies")
public class MovieCrudService {

    private Map<String, Movie> inventory = new HashMap<String, Movie>();

    @GET
    @Path("/getinfo")
    @Produces({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    public Movie movieByImdbId(@QueryParam("imdbId") String imdbId) {
        if (inventory.containsKey(imdbId)) {
            return inventory.get(imdbId);
        } else {
            return null;
        }
    }

    @POST
    @Path("/addmovie")
    @Consumes({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    public Response addMovie(Movie movie) {
        if (null != inventory.get(movie.getImdbId())) {
            return Response
                    .status(Response.Status.NOT_MODIFIED)
                    .entity("Movie is Already in the database.").build();
        }

        inventory.put(movie.getImdbId(), movie);
        return Response.status(Response.Status.CREATED).build();
    }
}
```

## 4. 总结

在此快速教程中，我们介绍了RESTEasy并用它构建了一个超级简单的API。