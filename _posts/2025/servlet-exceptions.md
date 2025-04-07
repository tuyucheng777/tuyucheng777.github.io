---
layout: post
title:  Jakarta EE Servlet异常处理
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 简介

在本教程中，我们将处理[Jakarta EE Servlet](https://www.baeldung.com/intro-to-servlets)应用程序中的异常-以便在发生错误时提供合理且预期的结果。

## 2. Jakarta EE Servlet异常

首先，我们将使用[API注解](https://tomcat.apache.org/tomcat-9.0-doc/servletapi/index.html)定义一个Servlet(有关详细信息，请参阅[Servlet简介](https://www.baeldung.com/intro-to-servlets))，并使用一个将引发异常的默认GET处理器：

```java
@WebServlet(urlPatterns = "/randomError")
public class RandomErrorServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        throw new IllegalStateException("Random error");
    }
}
```

## 3. 默认错误处理

现在让我们简单地将应用程序部署到Servlet容器中(我们假设应用程序在http://localhost:8080/jakarta-servlets下运行)。

当我们访问地址http://localhost:8080/jakarta-servlets/randomError时，我们将看到默认的Servlet错误处理：

![](/assets/images/2025/webmodules/servletexceptions01.png)

默认错误处理由Servlet容器提供，可以在[容器](https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Error_Report_Valve)或应用程序级别进行定制。

## 4. 自定义错误处理

我们可以使用web.xml文件描述符定义自定义错误处理，其中我们可以定义以下类型的策略：

- **状态码错误处理**：它允许我们将HTTP错误码([客户端](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_Client_errors)和[服务器](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#5xx_Server_errors))映射到静态HTML错误页面或错误处理Servlet
- **异常类型错误处理**：它允许我们将异常类型映射到静态HTML错误页面或错误处理Servlet

### 4.1 使用HTML页面处理状态码错误

我们可以在web.xml中为HTTP 404错误设置自定义错误处理策略：

```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <error-page>
        <error-code>404</error-code>
        <location>/error-404.html</location> <!-- /src/main/webapp/error-404.html-->
    </error-page>
</web-app>
```

现在，从浏览器访问http://localhost:8080/jakarta-servlets/invalid.html-获取静态HTML错误页面。

### 4.2 使用Servlet处理异常类型错误

我们可以在web.xml中为java.lang.Exception设置自定义错误处理策略：

```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <error-page>
        <exception-type>java.lang.Exception</exception-type>
        <location>/errorHandler</location>
    </error-page>
</web-app>
```

在ErrorHandlerServlet中，我们可以使用请求中提供的[错误属性](https://tomcat.apache.org/tomcat-7.0-doc/servletapi/constant-values.html)访问错误详细信息：

```java
@WebServlet(urlPatterns = "/errorHandler")
public class ErrorHandlerServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.setContentType("text/html; charset=utf-8");
        try (PrintWriter writer = resp.getWriter()) {
            writer.write("<html><head><title>Error description</title></head><body>");
            writer.write("<h2>Error description</h2>");
            writer.write("<ul>");
            Arrays.asList(
                            ERROR_STATUS_CODE,
                            ERROR_EXCEPTION_TYPE,
                            ERROR_MESSAGE)
                    .forEach(e ->
                            writer.write("<li>" + e + ":" + req.getAttribute(e) + " </li>")
                    );
            writer.write("</ul>");
            writer.write("</html></body>");
        }
    }
}
```

现在，我们可以访问http://localhost:8080/jakarta-servlets/randomError来查看自定义错误Servlet的运行情况。

注意：我们在web.xml中定义的异常类型太广泛，我们应该更详细地指定我们想要处理的所有异常。

我们还可以使用ErrorHandlerServlet组件中[容器提供的Servlet记录器](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/ServletContext.html#log-java.lang.String-)来记录其他详细信息：

```java
Exception exception = (Exception) req.getAttribute(ERROR_EXCEPTION);
if (IllegalArgumentException.class.isInstance(exception)) {
    getServletContext()
        .log("Error on an application argument", exception);
}
```

了解Servlet提供的日志记录机制之外的内容是很有价值的，请查看[Slf4j指南](https://www.baeldung.com/slf4j-with-log4j2-logback)了解更多详细信息。

## 5. 总结

在这篇简短的文章中，我们看到了Servlet应用程序中的默认错误处理和指定的自定义错误处理，无需添加外部组件或库。
