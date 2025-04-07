---
layout: post
title:  使用Java从HTTP请求获取客户端信息
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

[Web应用程序](https://www.baeldung.com/spring-boot-groovy-web-app)主要基于请求-响应模型，该模型使用HTTP协议描述客户端和[Web服务器](https://www.baeldung.com/java-servers)之间的数据交换，在接收或拒绝请求的服务器端，了解发出该请求的客户端非常重要。

在本教程中，我们将学习如何从HTTP请求中捕获客户端信息。

## 2. HTTP Request对象

在学习HTTP请求之前，我们首先应该了解[Servlet](https://www.baeldung.com/java-servlets-containers-intro)。Servlet是Java实现的一个基本部分，用于扩展Web开发的能力，以便处理HTTP请求并在响应中生成动态内容。

[HttpServletRequest](https://www.baeldung.com/spring-reading-httpservletrequest-multiple-times)是Java Servlet API中的一个接口，它表示客户端发出的HTTP请求。**HttpServletRequest对象在捕获有关客户端的重要信息方面非常方便**，提供了开箱即用的方法，例如getRemoteAddr()、getRemoteHost()、getHeader()和getRemoteUser()，这些方法有助于提取客户端信息。

### 2.1 获取客户端IP地址

我们可以使用getRemoteAddr()方法获取客户端的IP地址：

```java
String remoteAddr = request.getRemoteAddr(); // 198.167.0.1
```

值得注意的是，此方法检索服务器看到的IP地址，并且由于代理服务器、负载均衡器等因素，可能并不总是代表真正的客户端IP地址。

### 2.2 获取远程主机

我们可以使用getRemoteHost()方法获取客户端的主机名：

```java
String remoteHost = request.getRemoteHost(); // tuyucheng.com
```

### 2.3 获取远程用户

如果已经通过身份验证，我们可以使用getRemoteUser()方法获取客户端用户名：

```java
String remoteUser = request.getRemoteUser(); // tuyucheng
```

值得注意的是，如果客户端未经过身份验证，那么我们可能会得到null。

### 2.4 获取客户端标头

我们可以使用getHeader(headerName)方法读取客户端传递的标头值：

```java
String contentType = request.getHeader("content-type"); // application/json
```

**获取客户端信息的重要标头之一是User-Agent标头，它包括客户端的软件、系统等信息**。一些重要信息可能包括浏览器、操作系统、设备信息、插件、附加组件等。

以下是User-Agent字符串的示例：

```text
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36
```

我们可以使用HttpServletRequest提供的getHeader(String headerName)方法读取User-Agent标头。由于User-Agent字符串具有动态特性，因此 解析它本身就很复杂。不过，不同编程语言中都有可用的库来简化这项任务。对于Java生态系统，[uap-java](https://github.com/ua-parser/uap-java)是一个流行的选择。

除了上述方法之外，还有[其他方法](https://jakarta.ee/specifications/servlet/4.0/apidocs/javax/servlet/http/httpservletrequest)，例如getSessionID()，getMethod()，getRequestURL()等，根据使用情况可能会有帮助。

## 3. 提取客户端信息

如上一节所述，要解析User-Agent，我们可以使用[uap-java](https://mvnrepository.com/artifact/com.github.ua-parser/uap-java)库。为此，我们需要在pom.xml文件中添加以下XML代码片段：

```xml
<dependency> 
    <groupId>com.github.ua-parser</groupId> 
    <artifactId>uap-java</artifactId> 
    <version>1.5.4</version> 
</dependency>
```

配置完依赖后，让我们创建一个简单的AccountServlet，它充当客户端的HTTP端点并接收请求：

```java
@WebServlet(name = "AccountServlet", urlPatterns = "/account")
public class AccountServlet extends HttpServlet {
    public static final Logger log = LoggerFactory.getLogger(AccountServlet.class);

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        AccountLogic accountLogic = new AccountLogic();
        Map<String, String> clientInfo = accountLogic.getClientInfo(request);
        log.info("Request client info: {}, " + clientInfo);

        response.setStatus(HttpServletResponse.SC_OK);
    }
}
```

然后，我们可以将请求对象传递给AccountLogic，它从用户请求中提取客户端信息。然后，我们可以创建一个AccountLogic类，该类主要包含获取客户端信息的所有逻辑，我们可以使用前面讨论过的所有常见辅助方法：

```java
public class AccountLogic {
    public Map<String, String> getClientInfo(HttpServletRequest request) {
        String remoteAddr = request.getRemoteAddr();
        String remoteHost = request.getRemoteHost();
        String remoteUser = request.getRemoteUser();
        String contentType = request.getHeader("content-type");
        String userAgent = request.getHeader("user-agent");

        Parser uaParser = new Parser();
        Client client = uaParser.parse(userAgent);

        Map<String, String> clientInfo = new HashMap<>();
        clientInfo.put("os_family", client.os.family);
        clientInfo.put("device_family", client.device.family);
        clientInfo.put("userAgent_family", client.userAgent.family);
        clientInfo.put("remote_address", remoteAddr);
        clientInfo.put("remote_host", remoteHost);
        clientInfo.put("remote_user", remoteUser);
        clientInfo.put("content_type", contentType);
        return clientInfo;
    }
}
```

最后，我们编写一个简单的单元测试来验证功能：

```java
@Test
void givenMockHttpServletRequestWithHeaders_whenGetClientInfo_thenReturnsUserAGentInfo() {
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    when(request.getHeader("user-agent")).thenReturn("Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36, acceptLanguage:en-US,en;q=0.9");
    when(request.getHeader("content-type")).thenReturn("application/json");
    when(request.getRemoteAddr()).thenReturn("198.167.0.1");
    when(request.getRemoteHost()).thenReturn("tuyucheng.com");
    when(request.getRemoteUser()).thenReturn("tuyucheng");

    AccountLogic accountLogic = new AccountLogic();
    Map<String, String> clientInfo = accountLogic.getClientInfo(request);
    assertThat(clientInfo.get("os_family")).isEqualTo("Mac OS X");
    assertThat(clientInfo.get("device_family")).isEqualTo("Mac");
    assertThat(clientInfo.get("userAgent_family")).isEqualTo("Chrome");
    assertThat(clientInfo.get("content_type")).isEqualTo("application/json");
    assertThat(clientInfo.get("remote_user")).isEqualTo("tuyucheng");
    assertThat(clientInfo.get("remote_address")).isEqualTo("198.167.0.1");
    assertThat(clientInfo.get("remote_host")).isEqualTo("tuyucheng.com");
}
```

## 4. 总结

在本文中，我们了解了HttpServletRequest对象，它提供了有用的方法来捕获有关请求客户端的信息。我们还了解了User-Agent标头，它为客户端提供了系统级信息，例如浏览器系列、操作系统系列等。

随后我们还实现了从请求对象中捕获客户端信息的逻辑。