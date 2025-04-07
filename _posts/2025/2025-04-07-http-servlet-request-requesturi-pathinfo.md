---
layout: post
title:  HttpServletRequest中getRequestURI和getPathInfo的区别
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在本快速教程中，我们将讨论HttpServletRequest类中getRequestURI()和getPathInfo()之间的区别。

## 2. getRequestURI()和getPathInfo()之间的区别

**函数getRequestURI()返回完整的请求URI**，这包括部署文件夹和Servlet映射字符串；它还将返回所有额外的路径信息。

**getPathInfo()函数只返回传递给Servlet的路径**，如果没有传递额外的路径信息，该函数将返回null。

换句话说，**如果我们将应用程序部署在Web服务器的根目录中，并且请求映射到“/”的Servlet，则getRequestURI()和getPathInfo()都将返回相同的字符串**。否则，我们将获得不同的值。

## 3. 示例请求

为了更好地理解HttpServletRequest方法，假设我们有一个可以通过以下URL访问的[Servlet](https://www.baeldung.com/intro-to-servlets)：

```text
http://localhost:8080/deploy-folder/servlet-mapping
```

此请求将命中部署在“deploy-folder”内的Web应用程序中的“Servlet-mapping” Servlet，因此，如果我们针对此请求调用getRequestURI()和getPathInfo()，它们将返回不同的字符串。

让我们创建一个简单的doGet() Servlet方法：

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
    PrintWriter writer = response.getWriter();
    if ("getPathInfo".equals(request.getParameter("function")) {
        writer.println(request.getPathInfo());
    } else if ("getRequestURI".equals(request.getParameter("function")) {
        writer.println(request.getRequestURI());
    }
    writer.flush();
}
```

首先，让我们看一下通过[curl](https://www.baeldung.com/curl-rest)命令获取的getRequestURI请求的Servlet输出：

```shell
curl http://localhost:8080/deploy-folder/servlet-mapping/request-path?function=getRequestURI
/deploy-folder/servlet-mapping/request-path
```

类似地，让我们看一下getPathInfo的Servlet输出：

```shell
curl http://localhost:8080/deploy-folder/servlet-mapping/request-path?function=getPathInfo
/request-path
```

## 4. 总结

在本文中，我们解释了HttpServletRequest中getRequestURI()和getPathInfo()之间的区别，我们还通过示例Servlet和请求进行了演示。
