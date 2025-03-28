---
layout: post
title:  使用Java创建和配置Jetty 9服务器
category: libraries
copyright: libraries
excerpt: Jetty
---

## 1. 概述

在本文中，我们将讨论以编程方式创建和配置Jetty实例。

[Jetty](https://www.eclipse.org/jetty/)是一个HTTP服务器和Servlet容器，设计为轻量级且易于嵌入。我们将了解如何设置和配置一个或多个服务器实例。

## 2. Maven依赖

首先，我们要将[Jetty 9与以下Maven依赖](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-server)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-server</artifactId>
    <version>9.4.8.v20171121</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-webapp</artifactId>
    <version>9.4.8.v20171121</version>
</dependency>
```

## 3. 创建基本服务器

使用Jetty启动嵌入式服务器非常简单，只需编写：

```java
Server server = new Server();
server.start();
```

关闭它同样简单：

```java
server.stop();
```

## 4. 处理程序

现在我们的服务器已启动并运行，我们需要指示它如何处理传入的请求，这可以使用Handler接口执行。

我们可以自己创建一个，但是Jetty已经为最常见的用例提供了一组实现，让我们看看其中两个。

### 4.1 WebAppContext

WebAppContext类允许你将请求处理委托给现有的Web应用程序，该应用程序可以作为WAR文件路径或webapp文件夹路径提供。

如果我们想在“myApp”上下文中公开一个应用程序，我们可以这样写：

```java
Handler webAppHandler = new WebAppContext(webAppPath, "/myApp");
server.setHandler(webAppHandler);
```

### 4.2 HandlerCollection

对于复杂的应用程序，我们甚至可以使用HandlerCollection类指定多个处理程序。

假设我们已经实现了两个自定义处理程序，第一个处理程序仅执行日志记录操作，而第二个处理程序创建并向用户发送实际响应，我们希望按照此顺序使用这两个处理程序处理每个传入请求。

操作方法如下：

```java
Handler handlers = new HandlerCollection();
handlers.addHandler(loggingRequestHandler);
handlers.addHandler(customRequestHandler);
server.setHandler(handlers);
```

## 5. 连接器

我们要做的下一件事是配置服务器将监听的地址和端口并添加空闲超时。

Server类声明了两个便捷构造函数，可用于绑定到特定端口或地址。

**虽然在处理小型应用程序时这可能没问题，但如果我们想在不同的套接字上打开多个连接，这还不够**。

在这种情况下，Jetty提供了Connector接口，更具体地说是ServerConnector类，它允许定义各种连接配置参数：

```java
ServerConnector connector = new ServerConnector(server);
connector.setPort(80);
connector.setHost("169.20.45.12");
connector.setIdleTimeout(30000);
server.addConnector(connector);
```

使用此配置，服务器将监听169.20.45.12:80，在此地址上建立的每个连接的超时时间为30秒。

如果我们需要配置其他套接字，我们可以添加其他连接器。

## 6. 总结

在本快速教程中，我们重点介绍了如何使用Jetty设置嵌入式服务器，我们还了解了如何使用Handlers和Connectors进行进一步的配置。