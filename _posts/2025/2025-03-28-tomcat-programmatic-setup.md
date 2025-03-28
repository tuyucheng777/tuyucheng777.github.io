---
layout: post
title:  以编程方式创建、配置和运行Tomcat服务器
category: libraries
copyright: libraries
excerpt: Tomcat
---

## 1. 概述

在这篇简短的文章中，我们将以编程方式创建、配置和运行[Tomcat服务器](http://tomcat.apache.org/index.html)。

## 2. 设置

在开始之前，我们需要通过向pom.xml添加以下依赖来设置我们的Maven项目：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>tomcat-catalina</artifactId>
        <version>${tomcat.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>${apache.httpclient}</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>

```

这是[Maven Central链接](https://mvnrepository.com/artifact/junit/junit)，其中包含项目中使用的依赖的最新版本。

## 3. 初始化并配置Tomcat

我们先来谈谈初始化和配置Tomcat服务器所需的步骤。

### 3.1 创建Tomcat

我们可以通过以下简单的操作来创建一个实例：

```java
Tomcat tomcat = new Tomcat();
```

现在我们有了服务器，让我们来配置它。

### 3.2 配置Tomcat

我们将重点介绍如何启动并运行服务器，并添加Servlet和过滤器。

首先，我们需要配置端口、主机名和appBase(通常是Web应用)。为了达到我们的目的，我们将使用当前目录：

```java
tomcat.setPort(8080);
tomcat.setHostname("localhost");
String appBase = ".";
tomcat.getHost().setAppBase(appBase);
```

接下来，我们需要设置一个docBase(此Web应用程序的上下文根目录)：

```java
File docBase = new File(System.getProperty("java.io.tmpdir"));
Context context = tomcat.addContext("", docBase.getAbsolutePath());
```

至此，我们的Tomcat几乎可以正常运行。

接下来，我们将添加一个Servlet和一个过滤器并启动服务器以查看它是否正常运行。

### 3.3 将Servlet添加到Tomcat上下文

接下来，我们将向HttpServletResponse添加一个简单的文本，这是我们访问此Servlet的URL映射时将显示的文本。

首先让我们定义Servlet：

```java
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.setStatus(HttpServletResponse.SC_OK);
        resp.getWriter().write("test");
        resp.getWriter().flush();
        resp.getWriter().close();
    }
}
```

现在我们将这个Servlet添加到Tomcat服务器：

```java
Class servletClass = MyServlet.class;
Tomcat.addServlet(context, servletClass.getSimpleName(), servletClass.getName());
context.addServletMappingDecoded("/my-servlet/*", servletClass.getSimpleName());
```

### 3.4 向Tomcat上下文添加过滤器

接下来我们定义一个过滤器并将其添加到Tomcat：

```java
public class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {
        // ...
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        httpResponse.addHeader("myHeader", "myHeaderValue");
        chain.doFilter(request, httpResponse);
    }

    @Override
    public void destroy() {
        // ...
    }
}
```

将过滤器添加到上下文需要做更多的工作：

```java
Class filterClass = MyFilter.class;
FilterDef myFilterDef = new FilterDef();
myFilterDef.setFilterClass(filterClass.getName());
myFilterDef.setFilterName(filterClass.getSimpleName());
context.addFilterDef(myFilterDef);

FilterMap myFilterMap = new FilterMap();
myFilterMap.setFilterName(filterClass.getSimpleName());
myFilterMap.addURLPattern("/my-servlet/*");
context.addFilterMap(myFilterMap);
```

此时，我们应该已经将一个Servlet和一个过滤器添加到了Tomcat。

剩下要做的就是启动它并获取“test”页面并检查日志以查看过滤器是否有效。

## 4. 启动Tomcat

这是一个非常简单的操作，之后我们应该看到Tomcat正在运行：

```java
tomcat.start();
tomcat.getServer().await();
```

一旦启动，我们可以访问[http://localhost:8080/my-servlet](http://localhost:8080/my-servlet)并查看测试页面：

![](/assets/images/2025/libraries/tomcatprogrammaticsetup01.png)

如果我们查看日志，我们会看到类似这样的内容：

![](/assets/images/2025/libraries/tomcatprogrammaticsetup02.png)

这些日志表明Tomcat开始监听端口8080，并且我们的过滤器运行正常。

## 5. 总结

在本教程中，我们介绍了Tomcat服务器的基本程序设置。

我们研究了如何创建、配置和运行服务器，还研究了如何以编程方式向Tomcat上下文添加Servlet和Filter。