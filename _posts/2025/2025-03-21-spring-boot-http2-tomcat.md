---
layout: post
title:  在Spring Boot中使用Tomcat启用HTTP2
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

[HTTP/2](https://www.baeldung.com/cs/http-versions)是广泛使用的HTTP/1.1协议的后继者，它通过采用多路复用和标头压缩等新功能来提高Web性能。

在本教程中，我们将介绍如何配置Spring Boot应用程序以在嵌入式[Tomcat](https://www.baeldung.com/tomcat)服务器上启用HTTP/2。

## 2. HTTP/2

超文本传输协议(HTTP)是一种用于获取互联网资源的应用程序协议，HTTP/1.1于1997年1月发布，二十多年来一直服务于大多数Web。此版本确定了在某些情况下导致性能缓慢的问题。

HTTP/2通过以下特点克服了HTTP/1.1中的性能问题：

- 多路复用：HTTP/1.1使用多个连接发送多个请求；多路复用允许通过单个连接发送请求，从而减少资源消耗和延迟
- 压缩报头：如果不进行压缩，由于TCP启动缓慢，通常需要多次往返才能发送报头；压缩报头可以保证报头在更少的往返中发送，并且大多数情况下只需一次往返即可
- 二进制：HTTP/2以二进制格式发送数据，以减少解析开销，并且消息大小比使用文本编码数据的HTTP/1.1更小

## 3. 先决条件

HTTP/2可以以明文或TLS方式运行，大多数Web浏览器不支持明文HTTP/2，因此我们建议通过TLS运行。我们首先在嵌入式Web服务器上启用SSL。

让我们生成一个[keystore](https://www.baeldung.com/java-keystore#what-is-a-keystore)来存储用于SSL/TLS的密钥和证书，并将其放在嵌入式Tomcat中。我们将在控制台中运行以下keytool来生成它：

```shell
$ keytool -genkeypair -alias http2-alias -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore keystore.p12 -validity 3650
```

keytool将要求我们输入密码，稍后我们需要将其添加到Spring Boot配置中。完成此过程后，该工具将生成keystore.p12文件，我们将此文件复制到Spring Boot应用程序下的resources文件夹中。

现在，我们必须向application.yml添加以下属性以在嵌入式Tomcat中启用HTTPS：

```yaml
server:
    ssl:
        enabled: true
        key-store: classpath:keystore.p12
        key-store-password: <your-password>
        key-store-type: PKCS12
        key-alias: http2-alias
```

## 4. 检查响应中的HTTP协议版本

默认情况下，Spring Boot嵌入式Tomcat不会启用HTTP/2协议来处理请求。让我们创建一个简单的Spring Boot REST端点来验证它：

```java
@RestController
public class Http2Controller {
    @GetMapping("/http2/echo")
    public String getChatbotResponse(@RequestParam String message) {
        return message;
    }
}
```

该端点只是接收请求参数消息并将其在响应主体中发回。

一旦启动应用程序，我们就可以在控制台中使用–http2参数执行以下[curl](https://www.baeldung.com/curl-rest)命令，以通过HTTP/2调用端点：

```shell
$ curl -I --http2 http://localhost:8080/http2/echo?message=hello
```

-I参数向我们显示了其他信息，例如响应中的HTTP协议版本。从打印输出中，我们可以看到应用程序返回：

```text
HTTP/1.1 200
```

或者，我们可以使用[Postman](https://www.baeldung.com/java-postman)来验证协议版本。它从11.8版开始支持HTTP/2协议，其中发送HTTP请求的协议版本仍为默认的HTTP/1.1。我们可以在设置选项卡中更改协议版本：

![](/assets/images/2025/springboot/springboothttp2tomcat01.png)

一旦我们发送请求，我们就可以从Postman控制台中的原始日志中找到HTTP协议版本：

![](/assets/images/2025/springboot/springboothttp2tomcat02.png)

即使我们以HTTP/2发送请求，curl和Postman都会返回HTTP/1.1响应。

## 5. 在Spring Boot中启用HTTP/2

现在，让我们在嵌入式Tomcat中启用HTTP/2，有两种方法可以在Spring Boot应用程序中启用HTTP/2。

第一种方法是通过定义一个配置类将Http2Protocol类添加到Tomcat HTTP连接器：

```java
@Configuration
public class Http2Config {
    @Bean
    public WebServerFactoryCustomizer<TomcatServletWebServerFactory> getWebServerFactoryCustomizer() {
        return factory -> {
            Connector httpConnector = new Connector(TomcatServletWebServerFactory.DEFAULT_PROTOCOL);
            httpConnector.setPort(8080);
            factory.addConnectorCustomizers(connector -> connector.addUpgradeProtocol(new Http2Protocol()));
            factory.addAdditionalTomcatConnectors(httpConnector);
        };
    }
}
```

**该配置类定制了嵌入式Tomcat服务器，并通过向HTTP连接器添加Http2Protocol升级来启用HTTP/2支持**。

此配置除了现有的运行带SSL的HTTP/2的端口8443之外，还打开了一个在HTTP上运行的附加端口8080。如果我们不想公开附加端口，我们可以使用替代方法来启用HTTP/2。

第二个就更简单了，我们只需要在application.yml中将属性server.http2.enabled标记为true即可：

```yaml
server:
    http2:
        enabled: true
```

应用其中任一个后重新启动应用程序，我们就可以执行相同的curl命令来查询REST端点；我们将收到以下响应，表明HTTP/2已启用：

```text
HTTP/2 200
```

如果我们再次向Postman发送请求，我们将看到以下响应：

![](/assets/images/2025/springboot/springboothttp2tomcat03.png)

## 6. 总结

HTTP/1.1长期以来一直是传递HTTP请求的主导协议，而HTTP/2则提供了更好的资源效率和更低的延迟。这是一项巨大的进步，对现代Web应用程序而言是一次重大升级。

要使用HTTP/2协议运行Spring Boot应用程序，我们必须在Spring Boot应用程序中启用SSL和HTTP/2设置。